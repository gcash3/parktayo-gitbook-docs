# SECTION 3: API REFERENCE (Comprehensive)

## 3.1 Authentication API

### Base URL
```
Development: http://localhost:5000/api/v1
Production:  https://api.parktayo.com/api/v1
```

### Authentication Headers
```http
Authorization: Bearer <jwt_token>
Content-Type: application/json
```

---

### POST /auth/register
**Description:** Register a new user account

**Access:** Public

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "firstName": "Juan",
  "lastName": "Dela Cruz",
  "phoneNumber": "+639171234567",
  "role": "client"
}
```

**Validation Rules:**
- `email`: Valid email format, unique
- `password`: Minimum 8 characters, 1 uppercase, 1 number, 1 special char
- `phoneNumber`: Philippines format (+639XXXXXXXXX)
- `role`: "client" or "landlord"

**Success Response (201):**
```json
{
  "status": "success",
  "message": "User registered successfully",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "data": {
    "user": {
      "_id": "65a1b2c3d4e5f6789012345a",
      "email": "user@example.com",
      "firstName": "Juan",
      "lastName": "Dela Cruz",
      "phoneNumber": "+639171234567",
      "role": "client",
      "isEmailVerified": false,
      "isPhoneVerified": false,
      "createdAt": "2025-01-15T08:30:00.000Z"
    }
  }
}
```

**Error Responses:**

```json
// 400 - Validation Error
{
  "status": "error",
  "message": "Validation failed",
  "errors": [
    {
      "field": "email",
      "message": "Email already exists"
    },
    {
      "field": "password",
      "message": "Password must be at least 8 characters"
    }
  ]
}

// 500 - Server Error
{
  "status": "error",
  "message": "Registration failed. Please try again later."
}
```

**Code Example (JavaScript):**
```javascript
const register = async (userData) => {
  try {
    const response = await fetch('https://api.parktayo.com/api/v1/auth/register', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(userData)
    });

    const data = await response.json();

    if (data.status === 'success') {
      // Store token securely
      localStorage.setItem('token', data.token);
      // Navigate to dashboard
      return data.data.user;
    } else {
      throw new Error(data.message);
    }
  } catch (error) {
    console.error('Registration error:', error);
    throw error;
  }
};
```

**Code Example (Flutter/Dart):**
```dart
Future<User> register(Map<String, dynamic> userData) async {
  try {
    final response = await http.post(
      Uri.parse('${ApiConfig.baseUrl}/auth/register'),
      headers: {'Content-Type': 'application/json'},
      body: json.encode(userData),
    );

    final data = json.decode(response.body);

    if (response.statusCode == 201 && data['status'] == 'success') {
      // Store token securely
      await _storage.write(key: 'token', value: data['token']);
      return User.fromJson(data['data']['user']);
    } else {
      throw Exception(data['message']);
    }
  } catch (e) {
    throw Exception('Registration failed: $e');
  }
}
```

---

### POST /auth/login
**Description:** Authenticate user and receive JWT token

**Access:** Public

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!"
}
```

**Success Response (200):**
```json
{
  "status": "success",
  "message": "Login successful",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "data": {
    "user": {
      "_id": "65a1b2c3d4e5f6789012345a",
      "email": "user@example.com",
      "firstName": "Juan",
      "lastName": "Dela Cruz",
      "role": "client",
      "isEmailVerified": true,
      "isPhoneVerified": true,
      "profilePhoto": "https://cloudinary.com/...",
      "lastLogin": "2025-01-15T08:30:00.000Z"
    }
  }
}
```

**Error Responses:**
```json
// 401 - Invalid Credentials
{
  "status": "error",
  "message": "Invalid email or password"
}

// 403 - Account Suspended
{
  "status": "error",
  "message": "Your account has been suspended. Contact support."
}

// 429 - Too Many Attempts
{
  "status": "error",
  "message": "Too many login attempts. Please try again in 15 minutes."
}
```

---

### POST /auth/forgot-password
**Description:** Request password reset link

**Access:** Public

**Request Body:**
```json
{
  "email": "user@example.com"
}
```

**Success Response (200):**
```json
{
  "status": "success",
  "message": "Password reset link sent to your email"
}
```

---

### POST /auth/reset-password
**Description:** Reset password using token

**Access:** Public

**Request Body:**
```json
{
  "token": "reset_token_from_email",
  "newPassword": "NewSecurePass123!"
}
```

**Success Response (200):**
```json
{
  "status": "success",
  "message": "Password reset successfully"
}
```

---

### POST /auth/verify-email
**Description:** Verify email address

**Access:** Public

**Request Body:**
```json
{
  "token": "verification_token_from_email"
}
```

**Success Response (200):**
```json
{
  "status": "success",
  "message": "Email verified successfully"
}
```

---

### POST /auth/send-phone-verification
**Description:** Send OTP to phone number

**Access:** Authenticated

**Request Body:**
```json
{
  "phoneNumber": "+639171234567"
}
```

**Success Response (200):**
```json
{
  "status": "success",
  "message": "Verification code sent to +639171234567",
  "data": {
    "expiresIn": 300
  }
}
```

---

### POST /auth/verify-phone
**Description:** Verify phone number with OTP

**Access:** Authenticated

**Request Body:**
```json
{
  "phoneNumber": "+639171234567",
  "code": "123456"
}
```

**Success Response (200):**
```json
{
  "status": "success",
  "message": "Phone number verified successfully"
}
```

---

## 3.2 Parking Spaces API

### GET /parking-spaces
**Description:** Search parking spaces with filters

**Access:** Public (for viewing), Authenticated (for booking)

**Query Parameters:**
```
Required:
- latitude: 14.5995 (decimal degrees)
- longitude: 120.9842 (decimal degrees)

Optional:
- radius: 5 (km, default: 5, max: 50)
- minPrice: 30 (₱)
- maxPrice: 150 (₱)
- amenities: CCTV,Covered,24/7 (comma-separated)
- vehicleType: car|motorcycle|truck|van|suv
- availableSlots: 1 (minimum available slots)
- sortBy: distance|price|rating (default: distance)
- page: 1 (default: 1)
- limit: 20 (default: 20, max: 100)
```

**Example Request:**
```http
GET /api/v1/parking-spaces?latitude=14.5995&longitude=120.9842&radius=5&amenities=CCTV,Covered&sortBy=price&page=1&limit=20
```

**Success Response (200):**
```json
{
  "status": "success",
  "data": {
    "parkingSpaces": [
      {
        "_id": "65a1b2c3d4e5f6789012345b",
        "name": "SM North EDSA Parking - Level 3",
        "address": "EDSA corner North Avenue, Quezon City",
        "location": {
          "type": "Point",
          "coordinates": [120.9842, 14.5995]
        },
        "distance": 0.8,
        "pricing": {
          "pricePer3Hours": 75,
          "pricePerHour": 25,
          "dailyRate": 400,
          "overtimeRate": 15
        },
        "capacity": {
          "totalSlots": 10,
          "availableSlots": 7,
          "occupiedSlots": 3
        },
        "amenities": [
          "CCTV",
          "Security Guard",
          "Covered",
          "Well Lit",
          "24/7"
        ],
        "vehicleTypes": ["car", "suv", "van"],
        "photos": [
          "https://res.cloudinary.com/parktayo/image/upload/v1234567890/parking1.jpg",
          "https://res.cloudinary.com/parktayo/image/upload/v1234567890/parking2.jpg"
        ],
        "rating": {
          "average": 4.5,
          "count": 120
        },
        "operatingHours": {
          "monday": { "open": "00:00", "close": "23:59", "is24Hours": true },
          "tuesday": { "open": "00:00", "close": "23:59", "is24Hours": true }
        },
        "landlord": {
          "_id": "65a1b2c3d4e5f6789012345c",
          "name": "SM Parking Management",
          "rating": 4.7
        },
        "status": "approved",
        "isVerified": true
      }
    ],
    "pagination": {
      "currentPage": 1,
      "totalPages": 3,
      "totalResults": 45,
      "resultsPerPage": 20
    }
  }
}
```

**Code Example (Flutter):**
```dart
Future<List<ParkingSpace>> searchParkingSpaces({
  required double latitude,
  required double longitude,
  double radius = 5,
  String? amenities,
  String? sortBy,
}) async {
  final queryParams = {
    'latitude': latitude.toString(),
    'longitude': longitude.toString(),
    'radius': radius.toString(),
    if (amenities != null) 'amenities': amenities,
    if (sortBy != null) 'sortBy': sortBy,
  };

  final uri = Uri.parse('${ApiConfig.baseUrl}/parking-spaces')
      .replace(queryParameters: queryParams);

  final response = await http.get(uri);

  if (response.statusCode == 200) {
    final data = json.decode(response.body);
    return (data['data']['parkingSpaces'] as List)
        .map((json) => ParkingSpace.fromJson(json))
        .toList();
  } else {
    throw Exception('Failed to load parking spaces');
  }
}
```

---

### GET /parking-spaces/:id
**Description:** Get detailed information about a specific parking space

**Access:** Public

**URL Parameters:**
- `id`: Parking space ObjectId

**Success Response (200):**
```json
{
  "status": "success",
  "data": {
    "parkingSpace": {
      "_id": "65a1b2c3d4e5f6789012345b",
      "name": "SM North EDSA Parking - Level 3",
      "description": "Covered parking with 24/7 security",
      "address": "EDSA corner North Avenue, Quezon City",
      "location": {
        "type": "Point",
        "coordinates": [120.9842, 14.5995]
      },
      "pricing": {
        "pricePer3Hours": 75,
        "pricePerHour": 25,
        "dailyRate": 400,
        "overtimeRate": 15
      },
      "capacity": {
        "totalSlots": 10,
        "availableSlots": 7
      },
      "amenities": ["CCTV", "Security Guard", "Covered", "Well Lit", "24/7"],
      "vehicleTypes": ["car", "suv", "van"],
      "photos": ["https://..."],
      "operatingHours": {
        "monday": { "open": "00:00", "close": "23:59", "is24Hours": true }
      },
      "landlord": {
        "_id": "65a1b2c3d4e5f6789012345c",
        "firstName": "Juan",
        "lastName": "Dela Cruz",
        "rating": 4.7,
        "totalBookings": 350,
        "responseTime": "< 1 hour"
      },
      "statistics": {
        "totalBookings": 450,
        "averageRating": 4.5,
        "totalReviews": 120,
        "peakHours": ["09:00-12:00", "18:00-21:00"]
      },
      "recentReviews": [
        {
          "user": "Maria S.",
          "rating": 5,
          "comment": "Excellent parking space, very secure!",
          "date": "2025-01-10T14:30:00.000Z"
        }
      ]
    }
  }
}
```

---

### POST /parking-spaces/:id/check-availability
**Description:** Check if parking space is available for specific time range

**Access:** Public

**URL Parameters:**
- `id`: Parking space ObjectId

**Request Body:**
```json
{
  "startTime": "2025-01-15T10:00:00.000Z",
  "endTime": "2025-01-15T13:00:00.000Z"
}
```

**Success Response (200):**
```json
{
  "status": "success",
  "data": {
    "available": true,
    "availableSlots": 7,
    "totalSlots": 10,
    "occupiedSlots": 3,
    "conflicts": 3,
    "message": "7 slots available for the selected time",
    "conflictingBookings": [
      {
        "startTime": "2025-01-15T09:00:00.000Z",
        "endTime": "2025-01-15T12:00:00.000Z"
      }
    ]
  }
}
```

**Error Response (200 - No Slots):**
```json
{
  "status": "success",
  "data": {
    "available": false,
    "availableSlots": 0,
    "totalSlots": 10,
    "occupiedSlots": 10,
    "message": "No available slots for the selected time",
    "suggestedTimes": [
      {
        "startTime": "2025-01-15T13:00:00.000Z",
        "availableSlots": 8
      }
    ]
  }
}
```

---

### POST /parking-spaces (Landlord Only)
**Description:** Create a new parking space listing

**Access:** Authenticated (Landlord role)

**Request Body:**
```json
{
  "name": "My Parking Space",
  "address": "123 Main St, Quezon City",
  "location": {
    "latitude": 14.5995,
    "longitude": 120.9842
  },
  "description": "Secure covered parking near mall",
  "pricePer3Hours": 75,
  "dailyRate": 400,
  "overtimeRate": 15,
  "totalSlots": 5,
  "vehicleTypes": ["car", "suv"],
  "amenities": ["CCTV", "Covered", "Well Lit"],
  "operatingHours": {
    "monday": { "open": "06:00", "close": "22:00" },
    "tuesday": { "open": "06:00", "close": "22:00" }
  }
}
```

**Success Response (201):**
```json
{
  "status": "success",
  "message": "Parking space created successfully. Pending admin approval.",
  "data": {
    "parkingSpace": {
      "_id": "65a1b2c3d4e5f6789012345d",
      "status": "pending",
      "estimatedApprovalTime": "1-3 business days"
    }
  }
}
```

---

## 3.3 Bookings API

### POST /bookings
**Description:** Create a traditional booking (manual date/time selection)

**Access:** Authenticated

**Request Body:**
```json
{
  "parkingSpaceId": "65a1b2c3d4e5f6789012345b",
  "startTime": "2025-01-15T10:00:00.000Z",
  "endTime": "2025-01-15T13:00:00.000Z",
  "vehicleId": "65a1b2c3d4e5f6789012345e",
  "userNotes": "Near entrance please"
}
```

**Success Response (201):**
```json
{
  "status": "success",
  "message": "Booking created successfully",
  "data": {
    "booking": {
      "_id": "65a1b2c3d4e5f6789012345f",
      "bookingMode": "reservation",
      "status": "pending",
      "startTime": "2025-01-15T10:00:00.000Z",
      "endTime": "2025-01-15T13:00:00.000Z",
      "duration": 3,
      "parkingSpace": {
        "_id": "65a1b2c3d4e5f6789012345b",
        "name": "SM North EDSA Parking",
        "address": "EDSA corner North Avenue, Quezon City"
      },
      "vehicle": {
        "_id": "65a1b2c3d4e5f6789012345e",
        "plateNumber": "ABC 1234",
        "model": "Toyota Vios",
        "type": "car"
      },
      "pricing": {
        "baseParkingFee": 75.00,
        "dynamicParkingFee": 82.50,
        "serviceFee": 9.13,
        "totalAmount": 91.63,
        "demandFactor": 0.10,
        "breakdown": {
          "baseRate": "₱75.00 (3 hours)",
          "demandAdjustment": "+₱7.50 (10% demand)",
          "serviceFee": "₱9.13 (₱5 + 5%)",
          "total": "₱91.63"
        }
      },
      "payment": {
        "status": "held",
        "method": "wallet",
        "heldAmount": 91.63
      },
      "qrCode": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUg...",
      "userNotes": "Near entrance please",
      "createdAt": "2025-01-15T08:30:00.000Z"
    }
  }
}
```

**Error Responses:**
```json
// 400 - No Available Slots
{
  "status": "error",
  "message": "No available slots for the selected time",
  "data": {
    "availableSlots": 0,
    "suggestedTimes": [
      "2025-01-15T13:00:00.000Z"
    ]
  }
}

// 400 - Insufficient Balance
{
  "status": "error",
  "message": "Insufficient wallet balance",
  "data": {
    "required": 91.63,
    "available": 50.00,
    "shortfall": 41.63
  }
}

// 400 - Invalid Time
{
  "status": "error",
  "message": "Invalid booking time",
  "errors": [
    {
      "field": "startTime",
      "message": "Start time must be in the future"
    },
    {
      "field": "endTime",
      "message": "End time must be after start time"
    }
  ]
}
```

---

### POST /smart-booking
**Description:** Create a smart booking (ETA-based automatic reservation)

**Access:** Authenticated

**Request Body:**
```json
{
  "spaceId": "65a1b2c3d4e5f6789012345b",
  "destinationLat": 14.5995,
  "destinationLng": 120.9842,
  "userCurrentLat": 14.6022,
  "userCurrentLng": 120.9897,
  "vehicleId": "65a1b2c3d4e5f6789012345e",
  "preference": "balanced"
}
```

**Request Parameters:**
- `preference`: "fastest" | "balanced" | "cheapest"
  - **fastest**: Prioritize shortest ETA
  - **balanced**: Balance between ETA and price
  - **cheapest**: Prioritize lowest price

**Success Response (201):**
```json
{
  "status": "success",
  "message": "Smart booking created successfully",
  "data": {
    "booking": {
      "_id": "65a1b2c3d4e5f678901234560",
      "bookingMode": "book_now",
      "status": "accepted",
      "startTime": "2025-01-15T10:00:00.000Z",
      "endTime": "2025-01-15T10:40:00.000Z",
      "arrivalPrediction": {
        "realETAMinutes": 25,
        "gracePeriodMinutes": 15,
        "maxArrivalWindow": "2025-01-15T10:40:00.000Z",
        "estimatedArrival": "2025-01-15T10:25:00.000Z",
        "trafficCondition": "moderate",
        "route": {
          "distance": "3.5 km",
          "durationWithTraffic": "25 minutes",
          "polyline": "encoded_polyline_string"
        }
      },
      "locationTracking": {
        "enteredApproachingZone": false,
        "enteredArrivalZone": false,
        "enteredParkingZone": false,
        "monitoringActive": true
      },
      "parkingSpace": {
        "_id": "65a1b2c3d4e5f6789012345b",
        "name": "SM North EDSA Parking",
        "address": "EDSA corner North Avenue, Quezon City",
        "location": {
          "coordinates": [120.9842, 14.5995]
        }
      },
      "vehicle": {
        "_id": "65a1b2c3d4e5f6789012345e",
        "plateNumber": "ABC 1234",
        "model": "Toyota Vios"
      },
      "pricing": {
        "totalAmount": 91.63,
        "billingNote": "Final amount calculated after checkout based on actual parking duration"
      },
      "payment": {
        "status": "held",
        "heldAmount": 91.63
      },
      "qrCode": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUg...",
      "instructions": [
        "Start navigation to parking space",
        "You have until 10:40 AM to arrive",
        "System will track your location",
        "Check in automatically upon arrival",
        "Pay only for actual parking time"
      ]
    }
  }
}
```

---

### POST /smart-booking/update-location
**Description:** Update user location during smart booking journey

**Access:** Authenticated

**Request Body:**
```json
{
  "bookingId": "65a1b2c3d4e5f678901234560",
  "latitude": 14.6000,
  "longitude": 120.9850,
  "timestamp": "2025-01-15T10:10:00.000Z"
}
```

**Success Response (200):**
```json
{
  "status": "success",
  "data": {
    "locationUpdated": true,
    "distanceToParking": 1.2,
    "estimatedArrival": "2025-01-15T10:20:00.000Z",
    "zoneStatus": {
      "currentZone": "approaching",
      "enteredApproachingZone": true,
      "enteredArrivalZone": false,
      "enteredParkingZone": false
    }
  }
}
```

---

### GET /bookings/my-bookings
**Description:** Get user's bookings

**Access:** Authenticated

**Query Parameters:**
```
status: all|active|upcoming|past|cancelled
page: 1
limit: 20
sortBy: createdAt|startTime
order: desc|asc
```

**Success Response (200):**
```json
{
  "status": "success",
  "data": {
    "bookings": [
      {
        "_id": "65a1b2c3d4e5f678901234560",
        "parkingSpace": {
          "name": "SM North EDSA Parking",
          "address": "EDSA corner North Avenue, Quezon City",
          "photos": ["https://..."]
        },
        "startTime": "2025-01-15T10:00:00.000Z",
        "endTime": "2025-01-15T13:00:00.000Z",
        "status": "accepted",
        "bookingMode": "reservation",
        "pricing": {
          "totalAmount": 91.63
        },
        "payment": {
          "status": "held"
        },
        "qrCode": "data:image/png;base64,...",
        "canCheckIn": true,
        "canCancel": true,
        "canExtend": false
      }
    ],
    "pagination": {
      "currentPage": 1,
      "totalPages": 2,
      "totalResults": 25
    }
  }
}
```

---

### GET /bookings/:id
**Description:** Get detailed booking information

**Access:** Authenticated (own bookings only)

**Success Response (200):**
```json
{
  "status": "success",
  "data": {
    "booking": {
      "_id": "65a1b2c3d4e5f678901234560",
      "bookingMode": "reservation",
      "status": "accepted",
      "startTime": "2025-01-15T10:00:00.000Z",
      "endTime": "2025-01-15T13:00:00.000Z",
      "duration": 3,
      "parkingSpace": {
        "_id": "65a1b2c3d4e5f6789012345b",
        "name": "SM North EDSA Parking",
        "address": "EDSA corner North Avenue, Quezon City",
        "location": {
          "coordinates": [120.9842, 14.5995]
        },
        "photos": ["https://..."],
        "amenities": ["CCTV", "Covered"],
        "landlord": {
          "name": "Juan Dela Cruz",
          "phone": "+639171234567"
        }
      },
      "vehicle": {
        "plateNumber": "ABC 1234",
        "model": "Toyota Vios",
        "type": "car"
      },
      "pricing": {
        "baseParkingFee": 75.00,
        "dynamicParkingFee": 82.50,
        "serviceFee": 9.13,
        "totalAmount": 91.63,
        "breakdown": {
          "baseRate": "₱75.00 for 3 hours",
          "demandAdjustment": "+₱7.50 (10% peak demand)",
          "serviceFee": "₱9.13 (₱5 flat + 5% of ₱82.50)",
          "total": "₱91.63"
        }
      },
      "payment": {
        "status": "held",
        "method": "wallet",
        "heldAmount": 91.63
      },
      "qrCode": "data:image/png;base64,...",
      "timeline": [
        {
          "status": "created",
          "timestamp": "2025-01-15T08:30:00.000Z",
          "message": "Booking created"
        },
        {
          "status": "accepted",
          "timestamp": "2025-01-15T08:35:00.000Z",
          "message": "Landlord accepted booking"
        }
      ],
      "actions": {
        "canCheckIn": true,
        "canCancel": true,
        "canExtend": false,
        "canRate": false
      },
      "userNotes": "Near entrance please",
      "landlordNotes": null
    }
  }
}
```

---

### PUT /bookings/:id/cancel
**Description:** Cancel a booking

**Access:** Authenticated (own bookings only)

**Request Body:**
```json
{
  "reason": "Change of plans"
}
```

**Success Response (200):**
```json
{
  "status": "success",
  "message": "Booking cancelled successfully",
  "data": {
    "booking": {
      "_id": "65a1b2c3d4e5f678901234560",
      "status": "cancelled",
      "cancellationReason": "Change of plans",
      "cancelledAt": "2025-01-15T08:45:00.000Z"
    },
    "refund": {
      "amount": 91.63,
      "status": "processed",
      "processedAt": "2025-01-15T08:45:00.000Z"
    }
  }
}
```

**Error Response (400 - Cannot Cancel):**
```json
{
  "status": "error",
  "message": "Cannot cancel booking less than 1 hour before start time",
  "data": {
    "hoursUntilStart": 0.5,
    "minimumRequired": 1.0,
    "cancellationFee": 45.82
  }
}
```

---

### POST /bookings/:id/check-in
**Description:** Check in to parking space

**Access:** Authenticated

**Request Body:**
```json
{
  "location": {
    "latitude": 14.5995,
    "longitude": 120.9842
  }
}
```

**Success Response (200):**
```json
{
  "status": "success",
  "message": "Check-in successful",
  "data": {
    "booking": {
      "_id": "65a1b2c3d4e5f678901234560",
      "status": "parked",
      "checkInTime": "2025-01-15T10:05:00.000Z",
      "parkingSession": {
        "startTime": "2025-01-15T10:05:00.000Z",
        "estimatedEndTime": "2025-01-15T13:00:00.000Z"
      }
    }
  }
}
```

---

### POST /bookings/:id/checkout
**Description:** Checkout from parking space

**Access:** Authenticated

**Request Body:**
```json
{
  "location": {
    "latitude": 14.5995,
    "longitude": 120.9842
  }
}
```

**Success Response (200):**
```json
{
  "status": "success",
  "message": "Checkout successful",
  "data": {
    "booking": {
      "_id": "65a1b2c3d4e5f678901234560",
      "status": "completed",
      "checkOutTime": "2025-01-15T13:30:00.000Z",
      "parkingSession": {
        "startTime": "2025-01-15T10:05:00.000Z",
        "endTime": "2025-01-15T13:30:00.000Z",
        "actualDurationMinutes": 205,
        "actualDurationHours": 3.42
      }
    },
    "finalBilling": {
      "parkingDuration": "3 hours 25 minutes",
      "baseParkingFee": 75.00,
      "overtimeCharges": 15.00,
      "serviceFee": 9.50,
      "totalAmount": 99.50,
      "amountCharged": 99.50,
      "refund": 0.00,
      "breakdown": {
        "baseRate": "₱75.00 for first 3 hours",
        "overtime": "₱15.00 for 25 minutes (1 hour billed)",
        "serviceFee": "₱9.50 (₱5 + 5% of ₱90)",
        "total": "₱99.50"
      }
    },
    "receipt": {
      "receiptNumber": "RCP-2025011500001",
      "pdfUrl": "https://api.parktayo.com/receipts/RCP-2025011500001.pdf",
      "emailSent": true
    },
    "walletTransaction": {
      "captured": 99.50,
      "refunded": 0.00,
      "newBalance": 400.50
    }
  }
}
```

---

### POST /bookings/:id/extend
**Description:** Extend booking duration

**Access:** Authenticated

**Request Body:**
```json
{
  "additionalHours": 2,
  "newEndTime": "2025-01-15T15:00:00.000Z"
}
```

**Success Response (200):**
```json
{
  "status": "success",
  "message": "Booking extended successfully",
  "data": {
    "booking": {
      "_id": "65a1b2c3d4e5f678901234560",
      "endTime": "2025-01-15T15:00:00.000Z",
      "duration": 5
    },
    "additionalCharge": {
      "additionalHours": 2,
      "additionalAmount": 50.00,
      "totalHeld": 141.63
    }
  }
}
```

---

## 3.4 Wallet API

### GET /wallet
**Description:** Get wallet balance and information

**Access:** Authenticated

**Success Response (200):**
```json
{
  "status": "success",
  "data": {
    "wallet": {
      "_id": "65a1b2c3d4e5f678901234561",
      "userId": "65a1b2c3d4e5f6789012345a",
      "balance": 500.00,
      "availableBalance": 450.00,
      "heldAmount": 50.00,
      "pendingPayout": 0.00,
      "totalEarnings": 0.00,
      "currency": "PHP",
      "isActive": true,
      "lastTransactionDate": "2025-01-15T08:30:00.000Z"
    }
  }
}
```

---

### POST /wallet/top-up
**Description:** Add funds to wallet

**Access:** Authenticated

**Request Body:**
```json
{
  "amount": 500,
  "paymentMethod": "gcash",
  "referenceNumber": "GC-123456789"
}
```

**Payment Methods:**
- `gcash`: GCash
- `paymaya`: PayMaya
- `bank_transfer`: Bank Transfer
- `cash`: Cash (at selected locations)

**Success Response (200):**
```json
{
  "status": "success",
  "message": "Wallet top-up successful",
  "data": {
    "transaction": {
      "_id": "65a1b2c3d4e5f678901234562",
      "type": "top_up",
      "amount": 500.00,
      "status": "completed",
      "paymentMethod": "gcash",
      "referenceNumber": "GC-123456789",
      "processedAt": "2025-01-15T08:30:00.000Z"
    },
    "wallet": {
      "newBalance": 1000.00,
      "availableBalance": 950.00
    }
  }
}
```

---

### GET /wallet/transactions
**Description:** Get wallet transaction history

**Access:** Authenticated

**Query Parameters:**
```
type: all|top_up|booking_payment|refund|payout|earnings
status: all|pending|completed|failed
page: 1
limit: 20
startDate: 2025-01-01
endDate: 2025-01-31
```

**Success Response (200):**
```json
{
  "status": "success",
  "data": {
    "transactions": [
      {
        "_id": "65a1b2c3d4e5f678901234562",
        "type": "top_up",
        "amount": 500.00,
        "status": "completed",
        "description": "Wallet top-up via GCash",
        "paymentMethod": "gcash",
        "referenceNumber": "GC-123456789",
        "createdAt": "2025-01-15T08:30:00.000Z",
        "processedAt": "2025-01-15T08:30:00.000Z"
      },
      {
        "_id": "65a1b2c3d4e5f678901234563",
        "type": "booking_payment",
        "amount": -91.63,
        "status": "completed",
        "description": "Parking booking at SM North EDSA",
        "bookingId": "65a1b2c3d4e5f678901234560",
        "createdAt": "2025-01-15T13:30:00.000Z",
        "processedAt": "2025-01-15T13:30:00.000Z"
      },
      {
        "_id": "65a1b2c3d4e5f678901234564",
        "type": "refund",
        "amount": 45.82,
        "status": "completed",
        "description": "Booking cancellation refund",
        "bookingId": "65a1b2c3d4e5f678901234565",
        "createdAt": "2025-01-14T10:00:00.000Z",
        "processedAt": "2025-01-14T10:01:00.000Z"
      }
    ],
    "summary": {
      "totalTopUps": 1500.00,
      "totalSpending": 500.00,
      "totalRefunds": 100.00,
      "netBalance": 1100.00
    },
    "pagination": {
      "currentPage": 1,
      "totalPages": 3,
      "totalResults": 45
    }
  }
}
```

---

### POST /wallet/request-payout (Landlord Only)
**Description:** Request payout of earnings

**Access:** Authenticated (Landlord role)

**Request Body:**
```json
{
  "amount": 5000,
  "method": "bank_transfer",
  "bankDetails": {
    "accountName": "Juan Dela Cruz",
    "accountNumber": "1234567890",
    "bankName": "BDO",
    "branchCode": "0123"
  }
}
```

**Success Response (200):**
```json
{
  "status": "success",
  "message": "Payout request submitted successfully",
  "data": {
    "payout": {
      "_id": "65a1b2c3d4e5f678901234566",
      "amount": 5000.00,
      "status": "pending",
      "method": "bank_transfer",
      "estimatedProcessing": "1-3 business days",
      "requestedAt": "2025-01-15T08:30:00.000Z"
    }
  }
}
```

---

## 3.5 Rating & Review API

### POST /ratings
**Description:** Submit rating and review

**Access:** Authenticated

**Request Body:**
```json
{
  "bookingId": "65a1b2c3d4e5f678901234560",
  "parkingSpaceId": "65a1b2c3d4e5f6789012345b",
  "rating": 5,
  "comment": "Excellent parking space! Very secure and clean.",
  "aspects": {
    "cleanliness": 5,
    "security": 5,
    "accessibility": 4,
    "value": 5
  }
}
```

**Validation:**
- `rating`: 1-5 (required)
- `comment`: 10-500 characters (optional)
- `aspects`: Each 1-5 (optional)

**Success Response (201):**
```json
{
  "status": "success",
  "message": "Rating submitted successfully",
  "data": {
    "rating": {
      "_id": "65a1b2c3d4e5f678901234567",
      "rating": 5,
      "comment": "Excellent parking space! Very secure and clean.",
      "aspects": {
        "cleanliness": 5,
        "security": 5,
        "accessibility": 4,
        "value": 5
      },
      "createdAt": "2025-01-15T14:00:00.000Z"
    },
    "updatedAverageRating": 4.6
  }
}
```

---

### GET /ratings/parking-space/:id
**Description:** Get ratings for a parking space

**Access:** Public

**Query Parameters:**
```
page: 1
limit: 20
sortBy: recent|helpful|rating
```

**Success Response (200):**
```json
{
  "status": "success",
  "data": {
    "ratings": [
      {
        "_id": "65a1b2c3d4e5f678901234567",
        "user": {
          "firstName": "Maria",
          "lastName": "S.",
          "profilePhoto": "https://..."
        },
        "rating": 5,
        "comment": "Excellent parking space! Very secure and clean.",
        "aspects": {
          "cleanliness": 5,
          "security": 5,
          "accessibility": 4,
          "value": 5
        },
        "helpful": 12,
        "createdAt": "2025-01-15T14:00:00.000Z",
        "landlordResponse": null
      }
    ],
    "summary": {
      "averageRating": 4.6,
      "totalRatings": 120,
      "distribution": {
        "5": 80,
        "4": 25,
        "3": 10,
        "2": 3,
        "1": 2
      },
      "aspectAverages": {
        "cleanliness": 4.7,
        "security": 4.8,
        "accessibility": 4.3,
        "value": 4.5
      }
    },
    "pagination": {
      "currentPage": 1,
      "totalPages": 6,
      "totalResults": 120
    }
  }
}
```

---

## 3.6 Admin API

### GET /admin/dashboard
**Description:** Get admin dashboard statistics

**Access:** Authenticated (Admin role)

**Success Response (200):**
```json
{
  "status": "success",
  "data": {
    "statistics": {
      "users": {
        "total": 15000,
        "clients": 12000,
        "landlords": 3000,
        "newToday": 50,
        "newThisWeek": 300
      },
      "bookings": {
        "totalToday": 450,
        "activeNow": 120,
        "completedToday": 280,
        "cancelledToday": 15,
        "noShowsToday": 5
      },
      "revenue": {
        "today": 15000.00,
        "thisWeek": 85000.00,
        "thisMonth": 350000.00,
        "serviceFees": 17500.00
      },
      "parkingSpaces": {
        "total": 500,
        "active": 450,
        "pendingApproval": 35,
        "suspended": 15
      },
      "pendingActions": {
        "spaceApprovals": 35,
        "idVerifications": 20,
        "payouts": 15,
        "supportTickets": 8
      }
    },
    "recentActivity": [
      {
        "type": "new_user",
        "message": "New user registered: maria@example.com",
        "timestamp": "2025-01-15T14:30:00.000Z"
      }
    ],
    "alerts": [
      {
        "type": "warning",
        "message": "Server CPU usage at 85%",
        "severity": "medium"
      }
    ]
  }
}
```

---

### GET /admin/users
**Description:** Get all users with filters

**Access:** Authenticated (Admin role with user_management permission)

**Query Parameters:**
```
role: all|client|landlord
status: all|active|suspended
verified: all|true|false
search: email or name
page: 1
limit: 50
sortBy: createdAt|lastLogin|name
order: desc|asc
```

**Success Response (200):**
```json
{
  "status": "success",
  "data": {
    "users": [
      {
        "_id": "65a1b2c3d4e5f6789012345a",
        "email": "juan@example.com",
        "firstName": "Juan",
        "lastName": "Dela Cruz",
        "phoneNumber": "+639171234567",
        "role": "client",
        "isActive": true,
        "isEmailVerified": true,
        "isPhoneVerified": true,
        "profilePhoto": "https://...",
        "statistics": {
          "totalBookings": 45,
          "totalSpent": 5000.00,
          "averageRating": 4.8,
          "noShows": 0
        },
        "createdAt": "2024-06-15T08:30:00.000Z",
        "lastLogin": "2025-01-15T07:00:00.000Z"
      }
    ],
    "pagination": {
      "currentPage": 1,
      "totalPages": 300,
      "totalResults": 15000
    }
  }
}
```

---

### GET /admin/parking-spaces
**Description:** Get all parking spaces with admin details

**Access:** Authenticated (Admin role with space_management permission)

**Query Parameters:**
```
status: all|pending|approved|rejected|suspended
page: 1
limit: 50
```

**Success Response (200):**
```json
{
  "status": "success",
  "data": {
    "parkingSpaces": [
      {
        "_id": "65a1b2c3d4e5f6789012345b",
        "name": "SM North EDSA Parking",
        "address": "EDSA corner North Avenue, Quezon City",
        "landlord": {
          "_id": "65a1b2c3d4e5f6789012345c",
          "name": "Juan Dela Cruz",
          "email": "juan@example.com",
          "phone": "+639171234567",
          "isVerified": true
        },
        "pricing": {
          "pricePer3Hours": 75
        },
        "capacity": {
          "totalSlots": 10
        },
        "status": "pending",
        "isVerified": false,
        "photos": ["https://..."],
        "statistics": {
          "totalBookings": 0,
          "averageRating": 0,
          "totalRevenue": 0
        },
        "submittedAt": "2025-01-14T10:00:00.000Z"
      }
    ],
    "pagination": {
      "currentPage": 1,
      "totalPages": 10,
      "totalResults": 500
    }
  }
}
```

---

### PUT /admin/parking-spaces/:id/approve
**Description:** Approve pending parking space

**Access:** Authenticated (Admin role with space_management permission)

**Request Body:**
```json
{
  "notes": "Verified location and photos. Approved."
}
```

**Success Response (200):**
```json
{
  "status": "success",
  "message": "Parking space approved successfully",
  "data": {
    "parkingSpace": {
      "_id": "65a1b2c3d4e5f6789012345b",
      "status": "approved",
      "isVerified": true,
      "approvedBy": "65a1b2c3d4e5f678901234568",
      "approvedAt": "2025-01-15T14:30:00.000Z"
    }
  }
}
```

---

### GET /admin/payouts
**Description:** Get all payout requests

**Access:** Authenticated (Admin or Super Admin with financial_management permission)

**Query Parameters:**
```
status: all|pending|approved|completed|rejected
page: 1
limit: 20
```

**Success Response (200):**
```json
{
  "status": "success",
  "data": {
    "payouts": [
      {
        "_id": "65a1b2c3d4e5f678901234566",
        "landlord": {
          "_id": "65a1b2c3d4e5f6789012345c",
          "name": "Juan Dela Cruz",
          "email": "juan@example.com",
          "phone": "+639171234567"
        },
        "amount": 5000.00,
        "method": "bank_transfer",
        "bankDetails": {
          "accountName": "Juan Dela Cruz",
          "accountNumber": "****7890",
          "bankName": "BDO"
        },
        "status": "pending",
        "requestedAt": "2025-01-15T08:30:00.000Z",
        "earnings": {
          "totalBookings": 50,
          "totalEarnings": 7500.00,
          "availableBalance": 5000.00
        }
      }
    ],
    "summary": {
      "pendingAmount": 50000.00,
      "pendingCount": 10,
      "completedThisWeek": 85000.00
    },
    "pagination": {
      "currentPage": 1,
      "totalPages": 2,
      "totalResults": 25
    }
  }
}
```

---

### POST /admin/payouts/:id/approve
**Description:** Approve payout request

**Access:** Authenticated (Admin or Super Admin with financial_management permission)

**Request Body:**
```json
{
  "processingMethod": "bank_transfer",
  "referenceNumber": "BDO-20250115-001",
  "notes": "Processed via BDO bank transfer"
}
```

**Success Response (200):**
```json
{
  "status": "success",
  "message": "Payout approved and processed",
  "data": {
    "payout": {
      "_id": "65a1b2c3d4e5f678901234566",
      "status": "approved",
      "approvedBy": "65a1b2c3d4e5f678901234568",
      "approvedAt": "2025-01-15T14:30:00.000Z",
      "referenceNumber": "BDO-20250115-001"
    }
  }
}
```

---

### GET /admin/system-settings
**Description:** Get system settings (Super Admin Only)

**Access:** Authenticated (Super Admin only)

**Success Response (200):**
```json
{
  "status": "success",
  "data": {
    "settings": {
      "serviceFee": {
        "flatFee": 5,
        "percentage": 5
      },
      "booking": {
        "minimumHours": 3,
        "gracePeriodMinutes": 15,
        "autoRejectHours": 4,
        "cancellationWindowHours": 1
      },
      "pricing": {
        "overtimeRateDefault": 15,
        "maxDemandFactor": 0.5,
        "dynamicPricingEnabled": true
      },
      "payment": {
        "minimumTopUp": 100,
        "minimumPayout": 500,
        "payoutFrequency": "weekly"
      },
      "features": {
        "smartBookingEnabled": true,
        "aiSuggestionsEnabled": true,
        "qrCheckoutEnabled": true
      },
      "maintenance": {
        "maintenanceMode": false,
        "maintenanceMessage": null
      }
    }
  }
}
```

---

### PUT /admin/system-settings
**Description:** Update system settings (Super Admin Only)

**Access:** Authenticated (Super Admin only)

**Request Body:**
```json
{
  "serviceFee": {
    "flatFee": 5,
    "percentage": 5
  },
  "booking": {
    "minimumHours": 3,
    "gracePeriodMinutes": 15
  }
}
```

**Success Response (200):**
```json
{
  "status": "success",
  "message": "System settings updated successfully",
  "data": {
    "settings": {
      // Updated settings
    }
  }
}
```

---

## 3.7 Error Codes & Handling

### Standard Error Response Format
```json
{
  "status": "error",
  "message": "Human-readable error message",
  "code": "ERROR_CODE",
  "errors": [
    {
      "field": "fieldName",
      "message": "Field-specific error"
    }
  ],
  "timestamp": "2025-01-15T14:30:00.000Z",
  "path": "/api/v1/endpoint"
}
```

### HTTP Status Codes

| Code | Meaning | Description |
|------|---------|-------------|
| 200 | OK | Request successful |
| 201 | Created | Resource created successfully |
| 400 | Bad Request | Invalid request data |
| 401 | Unauthorized | Missing or invalid authentication |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource not found |
| 409 | Conflict | Resource conflict (e.g., duplicate) |
| 422 | Unprocessable Entity | Validation failed |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server error |
| 503 | Service Unavailable | Service temporarily unavailable |

### Common Error Codes

```
AUTH_001: Invalid credentials
AUTH_002: Token expired
AUTH_003: Token invalid
AUTH_004: Account suspended
AUTH_005: Email not verified

BOOKING_001: No available slots
BOOKING_002: Invalid booking time
BOOKING_003: Booking already exists
BOOKING_004: Cannot cancel (too late)
BOOKING_005: Insufficient balance

WALLET_001: Insufficient balance
WALLET_002: Invalid top-up amount
WALLET_003: Payment failed
WALLET_004: Payout limit not met

VALIDATION_001: Required field missing
VALIDATION_002: Invalid format
VALIDATION_003: Out of range

RATE_LIMIT_001: Too many requests
```

---
