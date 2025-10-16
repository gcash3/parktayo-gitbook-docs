# Services & Business Logic

## Smart Booking Service

**File:** `parktayo-backend/src/services/smartBookingService.js`

#### Key Functions:

```javascript
class SmartBookingService {
  /**
   * Calculate ETA using Google Routes API
   */
  async calculateETA(origin, destination) {
    // Input: { lat, lng } for both origin and destination
    // Output: { durationInTraffic, distance, route }

    const routes = await this.googleMapsService.getRoutes({
      origin: { latitude: origin.lat, longitude: origin.lng },
      destination: { latitude: destination.lat, longitude: destination.lng },
      travelMode: 'DRIVE',
      computeAlternativeRoutes: false,
      routingPreference: 'TRAFFIC_AWARE'
    });

    const route = routes[0];
    const etaMinutes = Math.ceil(route.duration / 60);

    return {
      etaMinutes,
      distance: route.distanceMeters,
      trafficCondition: route.polyline.trafficSpeed
    };
  }

  /**
   * Create smart booking with ETA-based window
   */
  async createSmartBooking(params) {
    const {
      userId,
      parkingSpaceId,
      userCurrentLocation,
      vehicleId
    } = params;

    // 1. Get parking space details
    const parkingSpace = await ParkingSpace.findById(parkingSpaceId);

    // 2. Calculate ETA
    const etaResult = await this.calculateETA(
      userCurrentLocation,
      {
        lat: parkingSpace.location.coordinates[1],
        lng: parkingSpace.location.coordinates[0]
      }
    );

    // 3. Calculate time window
    const gracePeriod = 15; // minutes
    const totalWindow = etaResult.etaMinutes + gracePeriod;

    // 4. Check availability
    const now = new Date();
    const endTime = new Date(now.getTime() + totalWindow * 60 * 1000);

    const conflicts = await Booking.checkConflicts(
      parkingSpaceId,
      now,
      endTime
    );

    const occupiedSlots = conflicts.length;
    const availableSlots = parkingSpace.totalSlots - occupiedSlots;

    if (availableSlots === 0) {
      throw new Error('No available slots for this time window');
    }

    // 5. Calculate pricing
    const pricing = await dynamicPricingService.calculatePricing({
      parkingSpaceId,
      startTime: now,
      duration: totalWindow / 60, // hours
      vehicleType: vehicleInfo.vehicleType
    });

    // 6. Hold wallet amount
    await walletService.holdAmount(userId, pricing.totalAmount);

    // 7. Create booking
    const booking = await Booking.create({
      userId,
      parkingSpaceId,
      landlordId: parkingSpace.landlordId,
      vehicleId,
      bookingMode: 'book_now',
      status: 'accepted', // Auto-approve smart bookings
      startTime: now,
      endTime,
      arrivalPrediction: {
        realETAMinutes: etaResult.etaMinutes,
        gracePeriodMinutes: gracePeriod,
        maxArrivalWindow: endTime,
        estimatedArrival: new Date(now.getTime() + etaResult.etaMinutes * 60 * 1000),
        trafficCondition: etaResult.trafficCondition
      },
      pricing
    });

    // 8. Schedule no-show check
    await noShowSchedulerService.scheduleCheck(booking._id, endTime);

    // 9. Notify landlord
    await notificationService.sendBookingNotification(
      parkingSpace.landlordId,
      'new_booking',
      booking
    );

    return booking;
  }

  /**
   * Update user location during journey
   */
  async updateUserLocation(bookingId, location) {
    const booking = await Booking.findById(bookingId);

    if (!booking || booking.status !== 'accepted') {
      return;
    }

    // Calculate distance to parking space
    const parkingSpace = await ParkingSpace.findById(booking.parkingSpaceId);
    const distance = this.calculateDistance(
      location,
      {
        lat: parkingSpace.location.coordinates[1],
        lng: parkingSpace.location.coordinates[0]
      }
    );

    // Update zone tracking
    const updates = {
      'locationTracking.lastKnownLocation': {
        type: 'Point',
        coordinates: [location.lng, location.lat]
      },
      'locationTracking.lastLocationUpdate': new Date()
    };

    // Zone detection
    if (distance <= 50) { // 50 meters - parking zone
      updates['locationTracking.enteredParkingZone'] = true;
    } else if (distance <= 200) { // 200 meters - arrival zone
      updates['locationTracking.enteredArrivalZone'] = true;
    } else if (distance <= 1000) { // 1km - approaching zone
      updates['locationTracking.enteredApproachingZone'] = true;
    }

    await Booking.updateOne({ _id: bookingId }, updates);
  }
}
```

## Dynamic Pricing Service

**File:** `parktayo-backend/src/services/dynamicPricingService.js`

```javascript
class DynamicPricingService {
  /**
   * Calculate dynamic pricing with demand factors
   */
  async calculatePricing(params) {
    const {
      parkingSpaceId,
      startTime,
      duration,
      vehicleType
    } = params;

    // 1. Get base pricing
    const parkingSpace = await ParkingSpace.findById(parkingSpaceId);
    const baseRate = parkingSpace.pricePer3Hours / 3; // per hour

    // 2. Vehicle type multiplier
    const vehicleMultipliers = {
      'truck': 1.5,      // HEAVY_VEHICLES
      'van': 1.2,
      'car': 1.0,        // MEDIUM_VEHICLES
      'suv': 1.1,
      'motorcycle': 0.7  // LIGHT_VEHICLES
    };

    const vehicleMultiplier = vehicleMultipliers[vehicleType] || 1.0;

    // 3. Calculate base parking fee
    const baseParkingFee = baseRate * duration * vehicleMultiplier;

    // 4. Calculate demand factor (0-50%)
    const demandFactor = await this.calculateDemandFactor(
      parkingSpaceId,
      startTime
    );

    // 5. Apply dynamic pricing
    const dynamicParkingFee = baseParkingFee * (1 + demandFactor);

    // 6. Calculate service fee (₱5 + 5%)
    const serviceFee = 5 + (dynamicParkingFee * 0.05);

    // 7. Total amount
    const totalAmount = dynamicParkingFee + serviceFee;

    // 8. Revenue split
    const landlordEarnings = baseParkingFee; // Landlord gets base rate only
    const platformEarnings = serviceFee + (dynamicParkingFee - baseParkingFee);

    return {
      baseParkingFee: Math.round(baseParkingFee * 100) / 100,
      dynamicParkingFee: Math.round(dynamicParkingFee * 100) / 100,
      serviceFee: Math.round(serviceFee * 100) / 100,
      totalAmount: Math.round(totalAmount * 100) / 100,
      demandFactor,
      landlordEarnings: Math.round(landlordEarnings * 100) / 100,
      platformEarnings: Math.round(platformEarnings * 100) / 100,
      vehicleCategory: this.getVehicleCategory(vehicleType)
    };
  }

  /**
   * Calculate demand factor based on occupancy
   */
  async calculateDemandFactor(parkingSpaceId, time) {
    const parkingSpace = await ParkingSpace.findById(parkingSpaceId);

    // Check bookings overlapping with requested time
    const oneHourLater = new Date(time.getTime() + 60 * 60 * 1000);
    const conflicts = await Booking.checkConflicts(
      parkingSpaceId,
      time,
      oneHourLater
    );

    const occupiedSlots = conflicts.length;
    const occupancyRate = occupiedSlots / parkingSpace.totalSlots;

    // Demand factor calculation:
    // 0-50% occupied = 0% increase
    // 51-70% occupied = 10% increase
    // 71-85% occupied = 25% increase
    // 86-100% occupied = 50% increase

    if (occupancyRate <= 0.50) return 0;
    if (occupancyRate <= 0.70) return 0.10;
    if (occupancyRate <= 0.85) return 0.25;
    return 0.50;
  }
}
```

## Wallet Service

**File:** `parktayo-backend/src/services/walletService.js`

```javascript
class WalletService {
  /**
   * Hold amount for booking (escrow)
   */
  async holdAmount(userId, amount) {
    const wallet = await Wallet.findOne({ userId });

    if (!wallet) {
      throw new Error('Wallet not found');
    }

    if (wallet.availableBalance < amount) {
      throw new Error('Insufficient balance');
    }

    // Move from available to held
    wallet.availableBalance -= amount;
    wallet.heldAmount += amount;
    await wallet.save();

    return wallet;
  }

  /**
   * Capture held amount (on successful checkout)
   */
  async captureAmount(userId, amount, bookingId) {
    const wallet = await Wallet.findOne({ userId });

    // Release from held amount
    wallet.heldAmount -= amount;
    wallet.balance -= amount;
    await wallet.save();

    // Create transaction record
    await Transaction.create({
      userId,
      walletId: wallet._id,
      type: 'booking_payment',
      amount,
      status: 'completed',
      bookingId
    });

    return wallet;
  }

  /**
   * Release held amount (on cancellation)
   */
  async releaseAmount(userId, amount, bookingId) {
    const wallet = await Wallet.findOne({ userId });

    // Move from held back to available
    wallet.heldAmount -= amount;
    wallet.availableBalance += amount;
    await wallet.save();

    // Create transaction record
    await Transaction.create({
      userId,
      walletId: wallet._id,
      type: 'refund',
      amount,
      status: 'completed',
      bookingId
    });

    return wallet;
  }

  /**
   * Transfer earnings to landlord
   */
  async transferEarnings(landlordId, amount, bookingId) {
    const wallet = await Wallet.findOne({ userId: landlordId });

    // Add to pending payout
    wallet.pendingPayout += amount;
    wallet.totalEarnings += amount;
    await wallet.save();

    // Create transaction record
    await Transaction.create({
      userId: landlordId,
      walletId: wallet._id,
      type: 'earnings',
      amount,
      status: 'completed',
      bookingId
    });

    return wallet;
  }
}
```

## No-Show Detection Service

**File:** `parktayo-backend/src/services/smartNoShowDetectionService.js`

```javascript
class SmartNoShowDetectionService {
  /**
   * Start monitoring user location for smart booking
   */
  async startMonitoring(booking) {
    const monitoringSession = {
      bookingId: booking._id,
      userId: booking.userId,
      parkingSpaceId: booking.parkingSpaceId,
      maxArrivalWindow: booking.arrivalPrediction.maxArrivalWindow,
      checkScheduledAt: booking.arrivalPrediction.maxArrivalWindow
    };

    // Store in memory or Redis for real-time tracking
    this.activeMonitoringSessions.set(
      booking._id.toString(),
      monitoringSession
    );

    // Schedule check at max arrival window
    this.scheduleNoShowCheck(booking._id, monitoringSession.checkScheduledAt);
  }

  /**
   * Check if user is a no-show (geolocation-based)
   */
  async checkNoShow(bookingId) {
    const booking = await Booking.findById(bookingId);

    if (!booking || booking.status !== 'accepted') {
      return; // Booking already processed
    }

    const tracking = booking.locationTracking;
    const now = new Date();

    // Decision logic:
    // 1. If entered parking zone (50m) → NOT no-show
    if (tracking.enteredParkingZone) {
      return { isNoShow: false, reason: 'User arrived at parking' };
    }

    // 2. If entered arrival zone (200m) → NOT no-show
    if (tracking.enteredArrivalZone) {
      return { isNoShow: false, reason: 'User near parking' };
    }

    // 3. If entered approaching zone and has recent location → NOT no-show
    if (tracking.enteredApproachingZone) {
      const lastUpdate = tracking.lastLocationUpdate;
      const timeSinceUpdate = (now - lastUpdate) / 1000 / 60; // minutes

      if (timeSinceUpdate <= 5) {
        return { isNoShow: false, reason: 'User approaching, stuck in traffic' };
      }
    }

    // 4. Otherwise → NO-SHOW
    return { isNoShow: true, reason: 'User did not arrive within time window' };
  }

  /**
   * Process no-show booking
   */
  async processNoShow(bookingId) {
    const booking = await Booking.findById(bookingId);

    // Mark as no-show
    booking.status = 'no_show';
    await booking.save();

    // Capture payment (charge user)
    await walletService.captureAmount(
      booking.userId,
      booking.pricing.totalAmount,
      bookingId
    );

    // Transfer earnings to landlord
    await walletService.transferEarnings(
      booking.landlordId,
      booking.pricing.landlordEarnings,
      bookingId
    );

    // Notify user
    await notificationService.sendBookingNotification(
      booking.userId,
      'no_show_penalty',
      booking
    );

    // Clean up monitoring session
    this.activeMonitoringSessions.delete(bookingId.toString());
  }
}
```

---
