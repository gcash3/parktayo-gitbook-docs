# Database Models

## User Model

```javascript
const userSchema = new mongoose.Schema({
  // Basic Information
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true
  },
  password: {
    type: String,
    required: true,
    select: false
  },
  firstName: String,
  lastName: String,
  phoneNumber: {
    type: String,
    unique: true,
    sparse: true
  },
  profilePhoto: String,

  // Role & Status
  role: {
    type: String,
    enum: ['client', 'landlord', 'admin'],
    default: 'client'
  },
  isActive: {
    type: Boolean,
    default: true
  },
  isEmailVerified: {
    type: Boolean,
    default: false
  },
  isPhoneVerified: {
    type: Boolean,
    default: false
  },

  // Verification
  emailVerificationToken: String,
  emailVerificationExpires: Date,
  phoneVerificationCode: String,
  phoneVerificationExpires: Date,

  // Firebase
  firebaseUID: String,
  fcmToken: String,

  // Timestamps
  createdAt: Date,
  updatedAt: Date,
  lastLogin: Date
});
```

## ParkingSpace Model

```javascript
const parkingSpaceSchema = new mongoose.Schema({
  // Owner Information
  landlordId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },

  // Location
  name: String,
  address: String,
  location: {
    type: {
      type: String,
      enum: ['Point'],
      default: 'Point'
    },
    coordinates: {
      type: [Number], // [longitude, latitude]
      required: true
    }
  },

  // Capacity
  totalSlots: {
    type: Number,
    default: 1,
    min: 1,
    max: 50
  },

  // Pricing (Per 3 Hours - Manila LGU Compliance)
  pricePer3Hours: {
    type: Number,
    required: true
  },
  dailyRate: Number,
  overtimeRate: {
    type: Number,
    default: 15 // ₱15 per hour
  },

  // Operating Hours
  operatingHours: {
    monday: { open: String, close: String, is24Hours: Boolean },
    tuesday: { open: String, close: String, is24Hours: Boolean },
    wednesday: { open: String, close: String, is24Hours: Boolean },
    thursday: { open: String, close: String, is24Hours: Boolean },
    friday: { open: String, close: String, is24Hours: Boolean },
    saturday: { open: String, close: String, is24Hours: Boolean },
    sunday: { open: String, close: String, is24Hours: Boolean }
  },

  // Amenities
  amenities: [{
    type: String,
    enum: [
      'CCTV',
      'Security Guard',
      'Covered',
      'Well Lit',
      '24/7',
      'EV Charging',
      'Accessible',
      'Restroom'
    ]
  }],

  // Vehicle Types
  vehicleTypes: [{
    type: String,
    enum: ['car', 'motorcycle', 'truck', 'van', 'suv']
  }],

  // Media
  photos: [String],

  // Status
  status: {
    type: String,
    enum: ['pending', 'approved', 'rejected', 'suspended'],
    default: 'pending'
  },
  isVerified: {
    type: Boolean,
    default: false
  },

  // Statistics
  totalBookings: { type: Number, default: 0 },
  averageRating: { type: Number, default: 0 },
  totalReviews: { type: Number, default: 0 },

  // Timestamps
  createdAt: Date,
  updatedAt: Date
});

// Geospatial index for location queries
parkingSpaceSchema.index({ location: '2dsphere' });
```

## Booking Model

```javascript
const bookingSchema = new mongoose.Schema({
  // Parties
  userId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  parkingSpaceId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'ParkingSpace',
    required: true
  },
  landlordId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  },

  // Vehicle
  vehicleId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Vehicle'
  },
  vehicleInfo: {
    plateNumber: String,
    vehicleModel: String,
    vehicleType: String
  },

  // Booking Type
  bookingMode: {
    type: String,
    enum: ['reservation', 'book_now'], // reservation = traditional, book_now = smart
    default: 'reservation'
  },

  // Time (Traditional Booking)
  startTime: {
    type: Date,
    required: true
  },
  endTime: {
    type: Date,
    required: true
  },
  duration: Number, // in hours

  // Smart Booking - Arrival Prediction
  arrivalPrediction: {
    realETAMinutes: Number,
    gracePeriodMinutes: Number,
    maxArrivalWindow: Date,
    estimatedArrival: Date,
    trafficCondition: String
  },

  // Smart Booking - Geolocation Tracking
  locationTracking: {
    enteredApproachingZone: {
      type: Boolean,
      default: false
    },
    enteredArrivalZone: {
      type: Boolean,
      default: false
    },
    enteredParkingZone: {
      type: Boolean,
      default: false
    },
    lastKnownLocation: {
      type: {
        type: String,
        enum: ['Point']
      },
      coordinates: [Number]
    },
    lastLocationUpdate: Date
  },

  // Parking Session (Usage-Based)
  parkingSession: {
    startTime: Date,
    endTime: Date,
    actualDurationMinutes: Number,
    overtimeMinutes: Number,
    isWithinWindow: Boolean
  },

  // Status
  status: {
    type: String,
    enum: [
      'pending',
      'accepted',
      'parked',
      'completed',
      'cancelled',
      'no_show',
      'expired'
    ],
    default: 'pending'
  },

  // Pricing
  pricing: {
    baseParkingFee: Number,
    dynamicParkingFee: Number,
    serviceFee: Number,
    totalAmount: Number,
    demandFactor: Number,
    landlordEarnings: Number,
    platformEarnings: Number,
    pricingModel: String
  },

  // Dynamic Pricing (New Structure)
  dynamicPricing: {
    vehicleType: String,
    vehicleCategory: String,
    customerBreakdown: {
      baseAmount: Number,
      dynamicAdjustments: Number,
      serviceFee: Number,
      totalAmount: Number
    },
    landlordBreakdown: {
      baseRate: Number,
      overtime: Number,
      total: Number
    },
    platformBreakdown: {
      serviceFee: Number,
      commission: Number,
      dynamicPricing: Number,
      total: Number
    }
  },

  // Payment
  paymentStatus: {
    type: String,
    enum: ['pending', 'held', 'captured', 'refunded'],
    default: 'pending'
  },
  paymentMethod: {
    type: String,
    default: 'wallet'
  },
  escrowTransactionId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'EscrowTransaction'
  },

  // Notes
  userNotes: String,
  landlordNotes: String,

  // QR Code
  qrCode: String,

  // Timestamps
  checkInTime: Date,
  checkOutTime: Date,
  createdAt: Date,
  updatedAt: Date
});

// Indexes for performance
bookingSchema.index({ userId: 1, status: 1 });
bookingSchema.index({ parkingSpaceId: 1, startTime: 1, endTime: 1 });
bookingSchema.index({ status: 1, createdAt: -1 });
```

## Wallet Model

```javascript
const walletSchema = new mongoose.Schema({
  userId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true,
    unique: true
  },

  // Balance
  balance: {
    type: Number,
    default: 0,
    min: 0
  },
  availableBalance: {
    type: Number,
    default: 0,
    min: 0
  },
  heldAmount: {
    type: Number,
    default: 0,
    min: 0
  },

  // Payout (for landlords)
  pendingPayout: {
    type: Number,
    default: 0
  },
  totalEarnings: {
    type: Number,
    default: 0
  },

  // Status
  isActive: {
    type: Boolean,
    default: true
  },

  // Timestamps
  createdAt: Date,
  updatedAt: Date,
  lastTransactionDate: Date
});
```

## Transaction Model

```javascript
const transactionSchema = new mongoose.Schema({
  userId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  walletId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Wallet'
  },

  // Transaction Details
  type: {
    type: String,
    enum: [
      'top_up',
      'booking_payment',
      'refund',
      'payout',
      'service_fee',
      'earnings'
    ],
    required: true
  },
  amount: {
    type: Number,
    required: true
  },

  // Status
  status: {
    type: String,
    enum: ['pending', 'completed', 'failed', 'cancelled'],
    default: 'pending'
  },

  // References
  bookingId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Booking'
  },
  referenceNumber: String,

  // Payment Method
  paymentMethod: {
    type: String,
    enum: ['gcash', 'paymaya', 'bank_transfer', 'wallet', 'cash']
  },

  // Description
  description: String,

  // Metadata
  metadata: mongoose.Schema.Types.Mixed,

  // Timestamps
  createdAt: Date,
  processedAt: Date
});
```

## Admin Model

```javascript
const adminSchema = new mongoose.Schema({
  email: {
    type: String,
    required: true,
    unique: true
  },
  password: {
    type: String,
    required: true,
    select: false
  },
  firstName: String,
  lastName: String,
  phoneNumber: String,

  // Role
  role: {
    type: String,
    default: 'admin'
  },

  // Admin Level Hierarchy
  adminLevel: {
    type: String,
    enum: ['super_admin', 'admin', 'moderator', 'support'],
    default: 'support'
  },

  // Permissions
  permissions: [{
    type: String,
    enum: [
      'user_management',
      'space_management',
      'booking_management',
      'financial_management',
      'content_management',
      'system_settings',
      'analytics_access',
      'support_tickets',
      'verification_approval',
      'emergency_actions'
    ]
  }],

  // Department
  department: {
    type: String,
    enum: [
      'operations',
      'customer_support',
      'finance',
      'tech',
      'marketing'
    ]
  },

  // Status
  isActive: {
    type: Boolean,
    default: true
  },

  // Timestamps
  createdAt: Date,
  updatedAt: Date,
  lastLogin: Date
});
```

---
