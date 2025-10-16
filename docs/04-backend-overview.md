# Backend API Overview

## Project Structure

```
parktayo-backend/
├── src/
│   ├── config/              # Configuration files
│   │   ├── database.js      # MongoDB connection
│   │   ├── firebase.js      # Firebase admin setup
│   │   ├── logger.js        # Winston logger
│   │   └── environment.js   # Environment variables
│   ├── models/              # Mongoose models
│   │   ├── User.js
│   │   ├── Admin.js
│   │   ├── ParkingSpace.js
│   │   ├── Booking.js
│   │   ├── Vehicle.js
│   │   ├── Wallet.js
│   │   ├── Transaction.js
│   │   └── Rating.js
│   ├── controllers/         # Request handlers
│   │   ├── authController.js
│   │   ├── bookingController.js
│   │   ├── smartBookingController.js
│   │   ├── parkingSpaceController.js
│   │   ├── adminController.js
│   │   └── walletController.js
│   ├── services/            # Business logic
│   │   ├── smartBookingService.js
│   │   ├── dynamicPricingService.js
│   │   ├── aiParkingSuggestionService.js
│   │   ├── bookingLifecycleService.js
│   │   ├── noShowSchedulerService.js
│   │   └── walletService.js
│   ├── routes/              # API routes
│   │   ├── auth.js
│   │   ├── bookings.js
│   │   ├── smartBooking.js
│   │   ├── parkingSpaces.js
│   │   ├── admin.js
│   │   └── legal.js
│   ├── middleware/          # Custom middleware
│   │   ├── auth.js          # JWT verification
│   │   ├── validation.js    # Input validation
│   │   ├── errorHandler.js  # Error handling
│   │   └── rateLimiter.js   # Rate limiting
│   ├── utils/               # Utility functions
│   │   ├── dateTime.js
│   │   ├── aiPricingUtils.js
│   │   └── timeValidationUtils.js
│   ├── data/                # Static data
│   │   ├── termsOfService.js
│   │   └── privacyPolicy.js
│   ├── templates/           # Email templates
│   │   └── receiptTemplate.html
│   └── server.js            # Express app entry point
├── tests/                   # Test files
├── .env.example             # Environment template
├── package.json
└── README.md
```

## Environment Variables

```bash
# Server
PORT=5000
NODE_ENV=production

# Database
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/parktayo

# Authentication
JWT_SECRET=your-super-secret-jwt-key
JWT_EXPIRES_IN=7d

# Google Maps
GOOGLE_MAPS_API_KEY=AIzaSyC...

# Firebase
FIREBASE_SERVICE_ACCOUNT=./firebase-service-account.json
FIREBASE_PROJECT_ID=parktayo-123456

# Cloudinary
CLOUDINARY_CLOUD_NAME=your-cloud-name
CLOUDINARY_API_KEY=123456789012345
CLOUDINARY_API_SECRET=your-api-secret
CLOUDINARY_URL=cloudinary://...

# SMS (Semaphore)
SEMAPHORE_API_KEY=your-semaphore-key

# Email (Optional)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password

# API Version
API_VERSION=v1

# Service Fee Configuration
PLATFORM_SERVICE_FEE_FLAT=5
PLATFORM_SERVICE_FEE_PERCENTAGE=5
```

## API Base URL Structure

```
Development: http://localhost:5000/api/v1
Production:  https://api.parktayo.com/api/v1
```

## Core API Modules

| Module | Route Prefix | Description |
|--------|-------------|-------------|
| Authentication | `/auth` | User registration, login, password reset |
| Bookings | `/bookings` | Traditional booking CRUD operations |
| Smart Booking | `/smart-booking` | ETA-based smart booking |
| Parking Spaces | `/parking-spaces` | Space listing, search, management |
| Wallet | `/wallet` | Top-up, transactions, balance |
| Vehicles | `/vehicles` | Vehicle registration and management |
| Admin | `/admin` | Admin-only operations |
| Legal | `/legal` | Terms, privacy policy |
| Ratings | `/ratings` | Reviews and ratings |
| Notifications | `/notifications` | Push notification management |

---
