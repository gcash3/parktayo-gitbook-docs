# SECTION 5: TESTING GUIDE

## 5.1 Testing Strategy Overview

### Testing Pyramid

```
                    ▲
                   / \
                  /   \
                 /  E2E \
                /_______\
               /         \
              /Integration\
             /   Tests     \
            /_______________\
           /                 \
          /   Unit Tests      \
         /                     \
        /_______________________\

Unit Tests (70%):
- Fast execution
- Test individual functions
- Mock dependencies
- High coverage

Integration Tests (20%):
- Test component interactions
- Database operations
- API endpoints
- Service integrations

E2E Tests (10%):
- Full user workflows
- Critical paths
- UI automation
- Performance testing
```

### Testing Coverage Goals

| Component | Unit | Integration | E2E | Target Coverage |
|-----------|------|-------------|-----|-----------------|
| Backend API | ✅ | ✅ | ✅ | 80% |
| User Mobile App | ✅ | ✅ | ✅ | 70% |
| Landlord App | ✅ | ✅ | ✅ | 70% |
| Admin Dashboard | ✅ | ✅ | ✅ | 75% |
| Database Models | ✅ | ✅ | - | 90% |

---

## 5.2 Backend Testing

### Unit Testing with Jest

#### Setup

**Install Dependencies:**
```bash
cd parktayo-backend
npm install --save-dev jest supertest mongodb-memory-server
```

**Configure Jest:**
```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'node',
  coveragePathIgnorePatterns: ['/node_modules/'],
  testMatch: ['**/__tests__/**/*.test.js'],
  collectCoverageFrom: [
    'src/**/*.js',
    '!src/server.js',
    '!src/config/**'
  ],
  coverageThreshold: {
    global: {
      branches: 70,
      functions: 70,
      lines: 70,
      statements: 70
    }
  },
  setupFilesAfterEnv: ['<rootDir>/tests/setup.js']
};
```

**Test Setup:**
```javascript
// tests/setup.js
const mongoose = require('mongoose');
const { MongoMemoryServer } = require('mongodb-memory-server');

let mongoServer;

beforeAll(async () => {
  mongoServer = await MongoMemoryServer.create();
  const mongoUri = mongoServer.getUri();

  await mongoose.connect(mongoUri, {
    useNewUrlParser: true,
    useUnifiedTopology: true
  });
});

afterAll(async () => {
  await mongoose.disconnect();
  await mongoServer.stop();
});

afterEach(async () => {
  const collections = mongoose.connection.collections;
  for (const key in collections) {
    await collections[key].deleteMany();
  }
});
```

---

### Testing Authentication

**tests/unit/auth.test.js:**
```javascript
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const User = require('../../src/models/User');
const authService = require('../../src/services/authService');

describe('Authentication Service', () => {
  describe('User Registration', () => {
    it('should create a new user with hashed password', async () => {
      const userData = {
        email: 'test@example.com',
        password: 'password123',
        firstName: 'Test',
        lastName: 'User',
        phoneNumber: '+639171234567',
        role: 'client'
      };

      const user = await authService.register(userData);

      expect(user).toBeDefined();
      expect(user.email).toBe(userData.email);
      expect(user.password).not.toBe(userData.password);
      expect(await bcrypt.compare(userData.password, user.password)).toBe(true);
    });

    it('should not allow duplicate email', async () => {
      const userData = {
        email: 'duplicate@example.com',
        password: 'password123',
        firstName: 'Test',
        lastName: 'User'
      };

      await authService.register(userData);

      await expect(authService.register(userData))
        .rejects
        .toThrow('Email already exists');
    });

    it('should validate email format', async () => {
      const userData = {
        email: 'invalid-email',
        password: 'password123'
      };

      await expect(authService.register(userData))
        .rejects
        .toThrow('Invalid email format');
    });
  });

  describe('User Login', () => {
    beforeEach(async () => {
      await authService.register({
        email: 'login@example.com',
        password: 'password123',
        firstName: 'Login',
        lastName: 'Test'
      });
    });

    it('should login with valid credentials', async () => {
      const result = await authService.login({
        email: 'login@example.com',
        password: 'password123'
      });

      expect(result).toBeDefined();
      expect(result.token).toBeDefined();
      expect(result.user.email).toBe('login@example.com');
    });

    it('should reject invalid password', async () => {
      await expect(authService.login({
        email: 'login@example.com',
        password: 'wrongpassword'
      })).rejects.toThrow('Invalid credentials');
    });

    it('should reject non-existent user', async () => {
      await expect(authService.login({
        email: 'notexist@example.com',
        password: 'password123'
      })).rejects.toThrow('Invalid credentials');
    });
  });

  describe('JWT Token', () => {
    it('should generate valid JWT token', () => {
      const userId = '507f1f77bcf86cd799439011';
      const token = authService.generateToken(userId, 'client');

      expect(token).toBeDefined();

      const decoded = jwt.verify(token, process.env.JWT_SECRET);
      expect(decoded.id).toBe(userId);
      expect(decoded.role).toBe('client');
    });

    it('should have expiration time', () => {
      const token = authService.generateToken('507f1f77bcf86cd799439011', 'client');
      const decoded = jwt.verify(token, process.env.JWT_SECRET);

      expect(decoded.exp).toBeDefined();
      expect(decoded.exp).toBeGreaterThan(decoded.iat);
    });
  });
});
```

---

### Testing Booking Service

**tests/unit/booking.test.js:**
```javascript
const bookingService = require('../../src/services/bookingService');
const ParkingSpace = require('../../src/models/ParkingSpace');
const Booking = require('../../src/models/Booking');
const User = require('../../src/models/User');
const Wallet = require('../../src/models/Wallet');

describe('Booking Service', () => {
  let user, parkingSpace, wallet;

  beforeEach(async () => {
    // Create test user
    user = await User.create({
      email: 'user@test.com',
      password: 'password123',
      firstName: 'Test',
      lastName: 'User',
      role: 'client'
    });

    // Create test parking space
    parkingSpace = await ParkingSpace.create({
      name: 'Test Parking',
      address: 'Test Address',
      location: {
        type: 'Point',
        coordinates: [120.9842, 14.5995]
      },
      landlordId: user._id,
      pricePer3Hours: 75,
      totalSlots: 5
    });

    // Create test wallet with sufficient balance
    wallet = await Wallet.create({
      userId: user._id,
      balance: 500,
      availableBalance: 500,
      heldAmount: 0
    });
  });

  describe('Create Booking', () => {
    it('should create booking with available slots', async () => {
      const bookingData = {
        userId: user._id,
        parkingSpaceId: parkingSpace._id,
        startTime: new Date('2025-01-20T10:00:00Z'),
        endTime: new Date('2025-01-20T13:00:00Z'),
        vehicleId: null
      };

      const booking = await bookingService.createBooking(bookingData);

      expect(booking).toBeDefined();
      expect(booking.status).toBe('pending');
      expect(booking.parkingSpaceId.toString()).toBe(parkingSpace._id.toString());
    });

    it('should reject booking with insufficient balance', async () => {
      // Update wallet to have low balance
      wallet.balance = 10;
      wallet.availableBalance = 10;
      await wallet.save();

      const bookingData = {
        userId: user._id,
        parkingSpaceId: parkingSpace._id,
        startTime: new Date('2025-01-20T10:00:00Z'),
        endTime: new Date('2025-01-20T13:00:00Z')
      };

      await expect(bookingService.createBooking(bookingData))
        .rejects
        .toThrow('Insufficient wallet balance');
    });

    it('should reject booking when no slots available', async () => {
      // Create bookings to fill all slots
      const startTime = new Date('2025-01-20T10:00:00Z');
      const endTime = new Date('2025-01-20T13:00:00Z');

      for (let i = 0; i < 5; i++) {
        await Booking.create({
          userId: user._id,
          parkingSpaceId: parkingSpace._id,
          startTime,
          endTime,
          status: 'accepted'
        });
      }

      const bookingData = {
        userId: user._id,
        parkingSpaceId: parkingSpace._id,
        startTime,
        endTime
      };

      await expect(bookingService.createBooking(bookingData))
        .rejects
        .toThrow('No available slots');
    });

    it('should calculate correct pricing', async () => {
      const bookingData = {
        userId: user._id,
        parkingSpaceId: parkingSpace._id,
        startTime: new Date('2025-01-20T10:00:00Z'),
        endTime: new Date('2025-01-20T13:00:00Z')
      };

      const booking = await bookingService.createBooking(bookingData);

      expect(booking.pricing).toBeDefined();
      expect(booking.pricing.baseParkingFee).toBe(75);
      expect(booking.pricing.serviceFee).toBeGreaterThan(0);
      expect(booking.pricing.totalAmount).toBeGreaterThan(75);
    });
  });

  describe('Check Availability', () => {
    it('should return correct available slots', async () => {
      const startTime = new Date('2025-01-20T10:00:00Z');
      const endTime = new Date('2025-01-20T13:00:00Z');

      // Create 2 bookings
      await Booking.create({
        userId: user._id,
        parkingSpaceId: parkingSpace._id,
        startTime,
        endTime,
        status: 'accepted'
      });

      await Booking.create({
        userId: user._id,
        parkingSpaceId: parkingSpace._id,
        startTime,
        endTime,
        status: 'accepted'
      });

      const availability = await bookingService.checkAvailability(
        parkingSpace._id,
        startTime,
        endTime
      );

      expect(availability.totalSlots).toBe(5);
      expect(availability.occupiedSlots).toBe(2);
      expect(availability.availableSlots).toBe(3);
      expect(availability.available).toBe(true);
    });

    it('should consider only overlapping bookings', async () => {
      // Booking before the time window
      await Booking.create({
        userId: user._id,
        parkingSpaceId: parkingSpace._id,
        startTime: new Date('2025-01-20T07:00:00Z'),
        endTime: new Date('2025-01-20T09:00:00Z'),
        status: 'accepted'
      });

      // Check availability for later time
      const availability = await bookingService.checkAvailability(
        parkingSpace._id,
        new Date('2025-01-20T10:00:00Z'),
        new Date('2025-01-20T13:00:00Z')
      );

      expect(availability.occupiedSlots).toBe(0);
      expect(availability.availableSlots).toBe(5);
    });
  });

  describe('Cancel Booking', () => {
    let booking;

    beforeEach(async () => {
      booking = await Booking.create({
        userId: user._id,
        parkingSpaceId: parkingSpace._id,
        startTime: new Date(Date.now() + 2 * 60 * 60 * 1000), // 2 hours from now
        endTime: new Date(Date.now() + 5 * 60 * 60 * 1000),
        status: 'pending',
        pricing: { totalAmount: 91.63 }
      });

      // Hold amount in wallet
      wallet.availableBalance -= 91.63;
      wallet.heldAmount += 91.63;
      await wallet.save();
    });

    it('should cancel booking and refund amount', async () => {
      const result = await bookingService.cancelBooking(
        booking._id,
        user._id,
        'Change of plans'
      );

      expect(result.booking.status).toBe('cancelled');
      expect(result.refund).toBeDefined();
      expect(result.refund.amount).toBe(91.63);

      // Check wallet
      const updatedWallet = await Wallet.findOne({ userId: user._id });
      expect(updatedWallet.heldAmount).toBe(0);
      expect(updatedWallet.availableBalance).toBeCloseTo(500, 2);
    });

    it('should not allow cancellation less than 1 hour before', async () => {
      // Update booking to start in 30 minutes
      booking.startTime = new Date(Date.now() + 30 * 60 * 1000);
      await booking.save();

      await expect(bookingService.cancelBooking(booking._id, user._id))
        .rejects
        .toThrow('Cannot cancel');
    });

    it('should not allow cancellation of completed booking', async () => {
      booking.status = 'completed';
      await booking.save();

      await expect(bookingService.cancelBooking(booking._id, user._id))
        .rejects
        .toThrow('Cannot cancel completed booking');
    });
  });
});
```

---

### Testing Dynamic Pricing

**tests/unit/dynamicPricing.test.js:**
```javascript
const dynamicPricingService = require('../../src/services/dynamicPricingService');
const ParkingSpace = require('../../src/models/ParkingSpace');
const Booking = require('../../src/models/Booking');

describe('Dynamic Pricing Service', () => {
  let parkingSpace;

  beforeEach(async () => {
    parkingSpace = await ParkingSpace.create({
      name: 'Test Parking',
      pricePer3Hours: 75,
      totalSlots: 10,
      location: {
        type: 'Point',
        coordinates: [120.9842, 14.5995]
      }
    });
  });

  describe('Calculate Pricing', () => {
    it('should calculate base pricing correctly', async () => {
      const pricing = await dynamicPricingService.calculatePricing({
        parkingSpaceId: parkingSpace._id,
        startTime: new Date(),
        duration: 3,
        vehicleType: 'car'
      });

      expect(pricing.baseParkingFee).toBe(75);
      expect(pricing.vehicleCategory).toBe('MEDIUM_VEHICLES');
    });

    it('should apply vehicle type multiplier', async () => {
      const carPricing = await dynamicPricingService.calculatePricing({
        parkingSpaceId: parkingSpace._id,
        startTime: new Date(),
        duration: 3,
        vehicleType: 'car'
      });

      const truckPricing = await dynamicPricingService.calculatePricing({
        parkingSpaceId: parkingSpace._id,
        startTime: new Date(),
        duration: 3,
        vehicleType: 'truck'
      });

      const motorcyclePricing = await dynamicPricingService.calculatePricing({
        parkingSpaceId: parkingSpace._id,
        startTime: new Date(),
        duration: 3,
        vehicleType: 'motorcycle'
      });

      expect(truckPricing.baseParkingFee).toBeGreaterThan(carPricing.baseParkingFee);
      expect(carPricing.baseParkingFee).toBeGreaterThan(motorcyclePricing.baseParkingFee);
    });

    it('should calculate service fee correctly', async () => {
      const pricing = await dynamicPricingService.calculatePricing({
        parkingSpaceId: parkingSpace._id,
        startTime: new Date(),
        duration: 3,
        vehicleType: 'car'
      });

      const expectedServiceFee = 5 + (pricing.dynamicParkingFee * 0.05);
      expect(pricing.serviceFee).toBeCloseTo(expectedServiceFee, 2);
    });
  });

  describe('Demand Factor Calculation', () => {
    it('should return 0 demand factor when occupancy is low', async () => {
      // No bookings, 0% occupancy
      const demandFactor = await dynamicPricingService.calculateDemandFactor(
        parkingSpace._id,
        new Date()
      );

      expect(demandFactor).toBe(0);
    });

    it('should increase demand factor with higher occupancy', async () => {
      const startTime = new Date();
      const endTime = new Date(startTime.getTime() + 3 * 60 * 60 * 1000);

      // Create bookings to occupy 8 out of 10 slots (80% occupancy)
      for (let i = 0; i < 8; i++) {
        await Booking.create({
          parkingSpaceId: parkingSpace._id,
          startTime,
          endTime,
          status: 'accepted'
        });
      }

      const demandFactor = await dynamicPricingService.calculateDemandFactor(
        parkingSpace._id,
        startTime
      );

      expect(demandFactor).toBeGreaterThan(0.2);
      expect(demandFactor).toBeLessThanOrEqual(0.5);
    });

    it('should return max demand factor when fully occupied', async () => {
      const startTime = new Date();
      const endTime = new Date(startTime.getTime() + 3 * 60 * 60 * 1000);

      // Fill all 10 slots
      for (let i = 0; i < 10; i++) {
        await Booking.create({
          parkingSpaceId: parkingSpace._id,
          startTime,
          endTime,
          status: 'accepted'
        });
      }

      const demandFactor = await dynamicPricingService.calculateDemandFactor(
        parkingSpace._id,
        startTime
      );

      expect(demandFactor).toBe(0.5); // Max 50%
    });
  });

  describe('Revenue Split', () => {
    it('should calculate correct landlord and platform earnings', async () => {
      const pricing = await dynamicPricingService.calculatePricing({
        parkingSpaceId: parkingSpace._id,
        startTime: new Date(),
        duration: 3,
        vehicleType: 'car'
      });

      // Landlord gets base parking fee
      expect(pricing.landlordEarnings).toBe(pricing.baseParkingFee);

      // Platform gets service fee + dynamic adjustments
      const expectedPlatformEarnings =
        pricing.serviceFee +
        (pricing.dynamicParkingFee - pricing.baseParkingFee);

      expect(pricing.platformEarnings).toBeCloseTo(expectedPlatformEarnings, 2);

      // Total should match
      const total = pricing.landlordEarnings + pricing.platformEarnings;
      expect(total).toBeCloseTo(pricing.totalAmount, 2);
    });
  });
});
```

---

### Integration Testing

**tests/integration/booking.integration.test.js:**
```javascript
const request = require('supertest');
const app = require('../../src/app');
const User = require('../../src/models/User');
const ParkingSpace = require('../../src/models/ParkingSpace');
const Wallet = require('../../src/models/Wallet');

describe('Booking API Integration Tests', () => {
  let authToken, user, parkingSpace;

  beforeEach(async () => {
    // Create and login user
    user = await User.create({
      email: 'integration@test.com',
      password: 'password123',
      firstName: 'Integration',
      lastName: 'Test',
      role: 'client'
    });

    const loginResponse = await request(app)
      .post('/api/v1/auth/login')
      .send({
        email: 'integration@test.com',
        password: 'password123'
      });

    authToken = loginResponse.body.token;

    // Create parking space
    parkingSpace = await ParkingSpace.create({
      name: 'Integration Test Parking',
      address: 'Test Address',
      location: {
        type: 'Point',
        coordinates: [120.9842, 14.5995]
      },
      landlordId: user._id,
      pricePer3Hours: 75,
      totalSlots: 5,
      status: 'approved'
    });

    // Create wallet with balance
    await Wallet.create({
      userId: user._id,
      balance: 500,
      availableBalance: 500
    });
  });

  describe('POST /api/v1/bookings', () => {
    it('should create booking successfully', async () => {
      const response = await request(app)
        .post('/api/v1/bookings')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          parkingSpaceId: parkingSpace._id,
          startTime: new Date(Date.now() + 2 * 60 * 60 * 1000).toISOString(),
          endTime: new Date(Date.now() + 5 * 60 * 60 * 1000).toISOString()
        });

      expect(response.status).toBe(201);
      expect(response.body.status).toBe('success');
      expect(response.body.data.booking).toBeDefined();
      expect(response.body.data.booking.status).toBe('pending');
    });

    it('should return 401 without authentication', async () => {
      const response = await request(app)
        .post('/api/v1/bookings')
        .send({
          parkingSpaceId: parkingSpace._id,
          startTime: new Date().toISOString(),
          endTime: new Date().toISOString()
        });

      expect(response.status).toBe(401);
    });

    it('should return 400 for invalid time range', async () => {
      const response = await request(app)
        .post('/api/v1/bookings')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          parkingSpaceId: parkingSpace._id,
          startTime: new Date().toISOString(),
          endTime: new Date(Date.now() - 1000).toISOString() // End before start
        });

      expect(response.status).toBe(400);
    });
  });

  describe('GET /api/v1/bookings/my-bookings', () => {
    beforeEach(async () => {
      // Create some bookings
      const Booking = require('../../src/models/Booking');
      await Booking.create({
        userId: user._id,
        parkingSpaceId: parkingSpace._id,
        startTime: new Date(Date.now() + 2 * 60 * 60 * 1000),
        endTime: new Date(Date.now() + 5 * 60 * 60 * 1000),
        status: 'pending'
      });
    });

    it('should return user bookings', async () => {
      const response = await request(app)
        .get('/api/v1/bookings/my-bookings')
        .set('Authorization', `Bearer ${authToken}`);

      expect(response.status).toBe(200);
      expect(response.body.status).toBe('success');
      expect(response.body.data.bookings).toBeInstanceOf(Array);
      expect(response.body.data.bookings.length).toBeGreaterThan(0);
    });

    it('should filter by status', async () => {
      const response = await request(app)
        .get('/api/v1/bookings/my-bookings?status=pending')
        .set('Authorization', `Bearer ${authToken}`);

      expect(response.status).toBe(200);
      expect(response.body.data.bookings.every(b => b.status === 'pending')).toBe(true);
    });
  });

  describe('POST /api/v1/parking-spaces/:id/check-availability', () => {
    it('should check availability correctly', async () => {
      const response = await request(app)
        .post(`/api/v1/parking-spaces/${parkingSpace._id}/check-availability`)
        .send({
          startTime: new Date(Date.now() + 2 * 60 * 60 * 1000).toISOString(),
          endTime: new Date(Date.now() + 5 * 60 * 60 * 1000).toISOString()
        });

      expect(response.status).toBe(200);
      expect(response.body.status).toBe('success');
      expect(response.body.data.available).toBe(true);
      expect(response.body.data.totalSlots).toBe(5);
      expect(response.body.data.availableSlots).toBe(5);
    });
  });
});
```

---

### API Load Testing

**tests/load/booking-load.test.js:**
```javascript
const autocannon = require('autocannon');

async function runLoadTest() {
  const result = await autocannon({
    url: 'http://localhost:5000',
    connections: 100, // concurrent connections
    duration: 30, // seconds
    pipelining: 1,
    requests: [
      {
        method: 'POST',
        path: '/api/v1/auth/login',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          email: 'load@test.com',
          password: 'password123'
        })
      }
    ]
  });

  console.log('Load Test Results:');
  console.log(`Requests: ${result.requests.total}`);
  console.log(`Throughput: ${result.throughput.mean} req/sec`);
  console.log(`Latency: ${result.latency.mean}ms (mean)`);
  console.log(`Errors: ${result.errors}`);
}

runLoadTest();
```

---

## 5.3 Frontend Testing (Flutter)

### Widget Testing

**test/widget/booking_screen_test.dart:**
```dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:parktayoflutter/screens/booking_screen.dart';
import 'package:parktayoflutter/models/parking_location.dart';

void main() {
  group('BookingScreen Widget Tests', () {
    late ParkingLocation testLocation;

    setUp(() {
      testLocation = ParkingLocation(
        id: '123',
        name: 'Test Parking',
        address: 'Test Address',
        latitude: 14.5995,
        longitude: 120.9842,
        pricePer3Hours: 75,
        totalSlots: 5,
        availableSlots: 3,
      );
    });

    testWidgets('should display parking space details', (WidgetTester tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: BookingScreen(parkingLocation: testLocation),
        ),
      );

      expect(find.text('Test Parking'), findsOneWidget);
      expect(find.text('Test Address'), findsOneWidget);
      expect(find.textContaining('₱75'), findsWidgets);
    });

    testWidgets('should show date time pickers', (WidgetTester tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: BookingScreen(parkingLocation: testLocation),
        ),
      );

      expect(find.byType(DateTimePicker), findsNWidgets(2));
    });

    testWidgets('should calculate duration correctly', (WidgetTester tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: BookingScreen(parkingLocation: testLocation),
        ),
      );

      // Set start time
      await tester.tap(find.byKey(Key('start-time-picker')));
      await tester.pumpAndSettle();

      // Set end time
      await tester.tap(find.byKey(Key('end-time-picker')));
      await tester.pumpAndSettle();

      // Check duration calculation
      expect(find.textContaining('Duration:'), findsOneWidget);
    });

    testWidgets('should show minimum billing warning for short duration',
        (WidgetTester tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: BookingScreen(parkingLocation: testLocation),
        ),
      );

      // Set duration less than 3 hours
      // ... set times ...

      await tester.pumpAndSettle();

      expect(find.textContaining('Minimum billing is 3 hours'), findsOneWidget);
    });

    testWidgets('should show availability warning when slots are low',
        (WidgetTester tester) async {
      final lowSlotLocation = testLocation.copyWith(availableSlots: 1);

      await tester.pumpWidget(
        MaterialApp(
          home: BookingScreen(parkingLocation: lowSlotLocation),
        ),
      );

      expect(find.textContaining('Only 1 slot'), findsOneWidget);
    });

    testWidgets('should disable confirm button when insufficient balance',
        (WidgetTester tester) async {
      // Mock wallet service with low balance
      await tester.pumpWidget(
        MaterialApp(
          home: BookingScreen(parkingLocation: testLocation),
        ),
      );

      final confirmButton = find.byKey(Key('confirm-booking-button'));
      expect(
        tester.widget<ElevatedButton>(confirmButton).enabled,
        isFalse
      );
    });
  });
}
```

---

### Integration Testing (Flutter)

**integration_test/booking_flow_test.dart:**
```dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';
import 'package:parktayoflutter/main.dart' as app;

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  group('Complete Booking Flow', () {
    testWidgets('should complete full booking process', (WidgetTester tester) async {
      app.main();
      await tester.pumpAndSettle();

      // Step 1: Login
      expect(find.text('Login'), findsOneWidget);

      await tester.enterText(find.byKey(Key('email-field')), 'test@example.com');
      await tester.enterText(find.byKey(Key('password-field')), 'password123');
      await tester.tap(find.byKey(Key('login-button')));
      await tester.pumpAndSettle();

      // Step 2: Search for parking
      expect(find.text('Find Parking'), findsOneWidget);

      await tester.tap(find.byIcon(Icons.search));
      await tester.pumpAndSettle();

      // Step 3: Select parking space
      await tester.tap(find.byType(ParkingCard).first);
      await tester.pumpAndSettle();

      // Step 4: View details and book
      expect(find.text('Book Parking'), findsOneWidget);
      await tester.tap(find.text('Book Parking'));
      await tester.pumpAndSettle();

      // Step 5: Select date and time
      await tester.tap(find.byKey(Key('start-time-picker')));
      await tester.pumpAndSettle();
      await tester.tap(find.text('OK'));
      await tester.pumpAndSettle();

      await tester.tap(find.byKey(Key('end-time-picker')));
      await tester.pumpAndSettle();
      await tester.tap(find.text('OK'));
      await tester.pumpAndSettle();

      // Step 6: Confirm booking
      await tester.tap(find.byKey(Key('confirm-booking-button')));
      await tester.pumpAndSettle(Duration(seconds: 5));

      // Step 7: Verify success
      expect(find.text('Booking Successful'), findsOneWidget);
    });
  });

  group('Smart Booking Flow', () => {
    testWidgets('should complete smart booking', (WidgetTester tester) async {
      app.main();
      await tester.pumpAndSettle();

      // Login
      // ... login steps ...

      // Tap "Book Now" button
      await tester.tap(find.text('Book Now'));
      await tester.pumpAndSettle();

      // Select destination
      await tester.enterText(
        find.byKey(Key('destination-search')),
        'SM North EDSA'
      );
      await tester.pumpAndSettle();

      await tester.tap(find.byType(LocationSuggestion).first);
      await tester.pumpAndSettle();

      // Confirm smart booking
      await tester.tap(find.byKey(Key('confirm-smart-booking-button')));
      await tester.pumpAndSettle(Duration(seconds: 5));

      // Verify ETA and booking details
      expect(find.textContaining('ETA:'), findsOneWidget);
      expect(find.text('Booking Created'), findsOneWidget);
    });
  });
}
```

---

## 5.4 E2E Testing

### Cypress E2E Testing (Admin Dashboard)

**Install Cypress:**
```bash
cd newparktayoadmin
npm install --save-dev cypress
```

**cypress/integration/admin/login.spec.js:**
```javascript
describe('Admin Login', () => {
  beforeEach(() => {
    cy.visit('http://localhost:3000/login');
  });

  it('should login successfully with valid credentials', () => {
    cy.get('[data-testid="email-input"]').type('admin@parktayo.com');
    cy.get('[data-testid="password-input"]').type('admin123');
    cy.get('[data-testid="login-button"]').click();

    cy.url().should('include', '/dashboard');
    cy.contains('Dashboard').should('be.visible');
  });

  it('should show error with invalid credentials', () => {
    cy.get('[data-testid="email-input"]').type('wrong@example.com');
    cy.get('[data-testid="password-input"]').type('wrongpassword');
    cy.get('[data-testid="login-button"]').click();

    cy.contains('Invalid credentials').should('be.visible');
  });

  it('should validate required fields', () => {
    cy.get('[data-testid="login-button"]').click();

    cy.contains('Email is required').should('be.visible');
    cy.contains('Password is required').should('be.visible');
  });
});
```

**cypress/integration/admin/parking-approval.spec.js:**
```javascript
describe('Parking Space Approval', () => {
  beforeEach(() => {
    cy.login('admin@parktayo.com', 'admin123');
    cy.visit('http://localhost:3000/parking-spaces');
  });

  it('should display pending parking spaces', () => {
    cy.contains('Pending Approvals').click();
    cy.get('[data-testid="pending-spaces-list"]').should('be.visible');
  });

  it('should approve parking space', () => {
    cy.contains('Pending Approvals').click();
    cy.get('[data-testid="space-card"]').first().click();

    cy.get('[data-testid="approve-button"]').click();
    cy.get('[data-testid="approval-notes"]').type('Verified and approved');
    cy.get('[data-testid="confirm-approval"]').click();

    cy.contains('Space approved successfully').should('be.visible');
  });

  it('should reject parking space with reason', () => {
    cy.contains('Pending Approvals').click();
    cy.get('[data-testid="space-card"]').first().click();

    cy.get('[data-testid="reject-button"]').click();
    cy.get('[data-testid="rejection-reason"]').type('Photos not clear');
    cy.get('[data-testid="confirm-rejection"]').click();

    cy.contains('Space rejected').should('be.visible');
  });
});
```

---

## 5.5 Performance Testing

### Backend Performance Benchmarks

**tests/performance/benchmark.js:**
```javascript
const Benchmark = require('benchmark');
const bookingService = require('../../src/services/bookingService');
const dynamicPricingService = require('../../src/services/dynamicPricingService');

const suite = new Benchmark.Suite();

suite
  .add('Booking Creation', async () => {
    await bookingService.createBooking({
      userId: 'test-user-id',
      parkingSpaceId: 'test-space-id',
      startTime: new Date(),
      endTime: new Date(Date.now() + 3 * 60 * 60 * 1000)
    });
  })
  .add('Availability Check', async () => {
    await bookingService.checkAvailability(
      'test-space-id',
      new Date(),
      new Date(Date.now() + 3 * 60 * 60 * 1000)
    );
  })
  .add('Dynamic Pricing Calculation', async () => {
    await dynamicPricingService.calculatePricing({
      parkingSpaceId: 'test-space-id',
      startTime: new Date(),
      duration: 3,
      vehicleType: 'car'
    });
  })
  .on('cycle', (event) => {
    console.log(String(event.target));
  })
  .on('complete', function() {
    console.log('Fastest is ' + this.filter('fastest').map('name'));
  })
  .run({ async: true });
```

---

## 5.6 Test Coverage Reports

### Generate Coverage Report

**Backend:**
```bash
npm test -- --coverage

# Generate HTML report
npm test -- --coverage --coverageReporters=html

# Open report
open coverage/index.html
```

**Expected Output:**
```
--------------------------|---------|----------|---------|---------|
File                      | % Stmts | % Branch | % Funcs | % Lines |
--------------------------|---------|----------|---------|---------|
All files                 |   82.45 |    78.23 |   85.67 |   83.12 |
 controllers              |   85.23 |    80.45 |   88.90 |   86.34 |
  authController.js       |   90.12 |    85.67 |   92.34 |   91.23 |
  bookingController.js    |   88.45 |    82.34 |   89.56 |   89.12 |
 services                 |   89.34 |    85.67 |   90.23 |   90.12 |
  bookingService.js       |   92.45 |    88.90 |   94.23 |   93.45 |
  dynamicPricingService.js|   87.23 |    83.45 |   88.67 |   88.34 |
 models                   |   95.67 |    92.34 |   96.78 |   96.23 |
--------------------------|---------|----------|---------|---------|
```

---

## 5.7 Continuous Integration Testing

**.github/workflows/test.yml:**
```yaml
name: Run Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [18.x]

    steps:
      - uses: actions/checkout@v2

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v2
        with:
          node-version: ${{ matrix.node-version }}

      - name: Install dependencies
        run: |
          cd parktayo-backend
          npm ci

      - name: Run linter
        run: |
          cd parktayo-backend
          npm run lint

      - name: Run unit tests
        run: |
          cd parktayo-backend
          npm test

      - name: Run integration tests
        run: |
          cd parktayo-backend
          npm run test:integration

      - name: Upload coverage reports
        uses: codecov/codecov-action@v2
        with:
          files: ./parktayo-backend/coverage/coverage-final.json
          flags: backend

      - name: Build application
        run: |
          cd parktayo-backend
          npm run build
```

---
