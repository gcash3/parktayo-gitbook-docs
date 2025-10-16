# API Endpoints

## Authentication Endpoints

#### POST /auth/register
Register new user account

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "SecurePass123",
  "firstName": "Juan",
  "lastName": "Dela Cruz",
  "phoneNumber": "+639171234567",
  "role": "client"
}
```

**Response (200):**
```json
{
  "status": "success",
  "message": "User registered successfully",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "data": {
    "user": {
      "_id": "65a1b2c3d4e5f6...",
      "email": "user@example.com",
      "firstName": "Juan",
      "lastName": "Dela Cruz",
      "role": "client",
      "isEmailVerified": false
    }
  }
}
```

#### POST /auth/login
Login to existing account

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "SecurePass123"
}
```

**Response (200):**
```json
{
  "status": "success",
  "message": "Login successful",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "data": {
    "user": {
      "_id": "65a1b2c3d4e5f6...",
      "email": "user@example.com",
      "firstName": "Juan",
      "role": "client"
    }
  }
}
```

## Parking Space Endpoints

#### GET /parking-spaces
Search parking spaces with filters

**Query Parameters:**
```
latitude: 14.5995
longitude: 120.9842
radius: 5 (km)
minPrice: 30
maxPrice: 100
amenities: CCTV,Covered
vehicleType: car
sortBy: distance / price / rating
```

**Response (200):**
```json
{
  "status": "success",
  "data": {
    "parkingSpaces": [
      {
        "_id": "65a1b2c3d4e5f6...",
        "name": "SM Mall Parking",
        "address": "123 EDSA, Quezon City",
        "location": {
          "type": "Point",
          "coordinates": [120.9842, 14.5995]
        },
        "pricePer3Hours": 75,
        "pricePerHour": 25,
        "totalSlots": 10,
        "availableSlots": 7,
        "amenities": ["CCTV", "Covered", "24/7"],
        "averageRating": 4.5,
        "totalReviews": 120,
        "distance": 0.8
      }
    ],
    "count": 15,
    "page": 1,
    "totalPages": 2
  }
}
```

#### POST /parking-spaces/:id/check-availability
Check slot availability for time range

**Request Body:**
```json
{
  "startTime": "2025-01-15T10:00:00.000Z",
  "endTime": "2025-01-15T13:00:00.000Z"
}
```

**Response (200):**
```json
{
  "status": "success",
  "data": {
    "available": true,
    "availableSlots": 7,
    "totalSlots": 10,
    "occupiedSlots": 3,
    "conflicts": 3,
    "message": "Parking space is available for the selected time"
  }
}
```

## Booking Endpoints

#### POST /bookings
Create traditional booking

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Request Body:**
```json
{
  "parkingSpaceId": "65a1b2c3d4e5f6...",
  "startTime": "2025-01-15T10:00:00.000Z",
  "endTime": "2025-01-15T13:00:00.000Z",
  "vehicleId": "65a1b2c3d4e5f7...",
  "userNotes": "Near entrance please"
}
```

**Response (201):**
```json
{
  "status": "success",
  "message": "Booking created successfully",
  "data": {
    "booking": {
      "_id": "65a1b2c3d4e5f8...",
      "bookingMode": "reservation",
      "status": "pending",
      "startTime": "2025-01-15T10:00:00.000Z",
      "endTime": "2025-01-15T13:00:00.000Z",
      "duration": 3,
      "pricing": {
        "baseParkingFee": 75,
        "dynamicParkingFee": 82.50,
        "serviceFee": 9.13,
        "totalAmount": 91.63,
        "demandFactor": 0.10
      },
      "paymentStatus": "held"
    }
  }
}
```

#### POST /smart-booking
Create smart booking (ETA-based)

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Request Body:**
```json
{
  "spaceId": "65a1b2c3d4e5f6...",
  "destinationLat": 14.5995,
  "destinationLng": 120.9842,
  "userCurrentLat": 14.6022,
  "userCurrentLng": 120.9897,
  "vehicleId": "65a1b2c3d4e5f7...",
  "preference": "balanced"
}
```

**Response (201):**
```json
{
  "status": "success",
  "message": "Smart booking created successfully",
  "data": {
    "booking": {
      "_id": "65a1b2c3d4e5f8...",
      "bookingMode": "book_now",
      "status": "accepted",
      "arrivalPrediction": {
        "realETAMinutes": 25,
        "gracePeriodMinutes": 15,
        "maxArrivalWindow": "2025-01-15T10:40:00.000Z",
        "estimatedArrival": "2025-01-15T10:25:00.000Z",
        "trafficCondition": "moderate"
      },
      "startTime": "2025-01-15T10:00:00.000Z",
      "endTime": "2025-01-15T10:40:00.000Z",
      "pricing": {
        "totalAmount": 91.63
      }
    }
  }
}
```

#### GET /bookings/my-bookings
Get user's bookings

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Query Parameters:**
```
status: active / upcoming / past
page: 1
limit: 10
```

**Response (200):**
```json
{
  "status": "success",
  "data": {
    "bookings": [
      {
        "_id": "65a1b2c3d4e5f8...",
        "parkingSpace": {
          "name": "SM Mall Parking",
          "address": "123 EDSA, Quezon City"
        },
        "startTime": "2025-01-15T10:00:00.000Z",
        "endTime": "2025-01-15T13:00:00.000Z",
        "status": "accepted",
        "pricing": {
          "totalAmount": 91.63
        },
        "qrCode": "data:image/png;base64,..."
      }
    ],
    "count": 5,
    "page": 1,
    "totalPages": 1
  }
}
```

## Wallet Endpoints

#### GET /wallet
Get wallet balance

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Response (200):**
```json
{
  "status": "success",
  "data": {
    "wallet": {
      "_id": "65a1b2c3d4e5f9...",
      "balance": 500.00,
      "availableBalance": 450.00,
      "heldAmount": 50.00,
      "pendingPayout": 0,
      "totalEarnings": 0
    }
  }
}
```

#### POST /wallet/top-up
Add funds to wallet

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Request Body:**
```json
{
  "amount": 500,
  "paymentMethod": "gcash",
  "referenceNumber": "GC-123456789"
}
```

**Response (200):**
```json
{
  "status": "success",
  "message": "Wallet topped up successfully",
  "data": {
    "transaction": {
      "_id": "65a1b2c3d4e5fa...",
      "type": "top_up",
      "amount": 500,
      "status": "completed",
      "referenceNumber": "GC-123456789"
    },
    "newBalance": 1000.00
  }
}
```

## Dynamic Pricing Endpoints

#### GET /smart-booking/dynamic-pricing
Calculate dynamic pricing

**Query Parameters:**
```
spaceId: 65a1b2c3d4e5f6...
bookingTime: 2025-01-15T10:00:00.000Z
duration: 3
vehicleType: car
```

**Response (200):**
```json
{
  "status": "success",
  "data": {
    "pricing": {
      "baseParkingFee": 75.00,
      "dynamicParkingFee": 82.50,
      "serviceFee": 9.13,
      "totalPrice": 91.63,
      "demandFactor": 0.10,
      "vehicleCategory": "MEDIUM_VEHICLES",
      "breakdown": {
        "baseRate": "₱75.00 (3 hours)",
        "demandAdjustment": "+₱7.50 (10% demand)",
        "serviceFee": "₱9.13 (₱5 + 5%)",
        "total": "₱91.63"
      }
    }
  }
}
```

## Legal Document Endpoints

#### GET /legal/terms
Get Terms of Service

**Query Parameters:**
```
format: json / html / text
```

**Response (200):**
```json
{
  "status": "success",
  "data": {
    "title": "Terms of Service",
    "version": "1.0.0",
    "effectiveDate": "2025-01-01",
    "lastUpdated": "2025-01-01",
    "content": "# TERMS OF SERVICE\n\n..."
  }
}
```

#### GET /legal/privacy
Get Privacy Policy

**Query Parameters:**
```
format: json / html / text
```

**Response (200):**
```json
{
  "status": "success",
  "data": {
    "title": "Privacy Policy",
    "version": "1.0.0",
    "effectiveDate": "2025-01-01",
    "lastUpdated": "2025-01-01",
    "content": "# PRIVACY POLICY\n\n..."
  }
}
```

## Admin Endpoints

#### GET /admin/system-settings
Get system settings (Super Admin Only)

**Headers:**
```
Authorization: Bearer <admin_jwt_token>
```

**Response (200):**
```json
{
  "status": "success",
  "data": {
    "settings": {
      "serviceFeeFlat": 5,
      "serviceFeePercentage": 5,
      "minimumBookingHours": 3,
      "gracePeriodMinutes": 15,
      "overtimeRatePerHour": 15,
      "demandFactorMax": 0.5
    }
  }
}
```

#### GET /admin/payouts
Get pending payouts

**Headers:**
```
Authorization: Bearer <admin_jwt_token>
```

**Query Parameters:**
```
status: pending / completed / rejected
page: 1
limit: 20
```

**Response (200):**
```json
{
  "status": "success",
  "data": {
    "payouts": [
      {
        "_id": "65a1b2c3d4e5fb...",
        "landlord": {
          "name": "Juan Dela Cruz",
          "email": "juan@example.com"
        },
        "amount": 5000.00,
        "status": "pending",
        "requestDate": "2025-01-15T00:00:00.000Z"
      }
    ],
    "count": 10,
    "totalAmount": 50000.00
  }
}
```

---
