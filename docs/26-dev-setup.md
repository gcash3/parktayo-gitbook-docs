# Development Setup

## Development Environment Setup

### Prerequisites

#### System Requirements
```
Operating System:
- Windows 10/11
- macOS 10.15+
- Ubuntu 20.04+

Memory: 8GB RAM minimum (16GB recommended)
Storage: 50GB free space
```

#### Required Software

**1. Node.js & npm**
```bash
# Install Node.js v18.x (LTS)
# Download from: https://nodejs.org/

# Verify installation
node --version  # Should output: v18.x.x
npm --version   # Should output: 9.x.x
```

**2. MongoDB**
```bash
# Option A: MongoDB Atlas (Cloud - Recommended)
# Sign up at: https://www.mongodb.com/cloud/atlas
# Create free M0 cluster
# Get connection string

# Option B: Local MongoDB
# Download from: https://www.mongodb.com/try/download/community
# Install and start service

# Verify installation
mongosh --version
```

**3. Git**
```bash
# Download from: https://git-scm.com/downloads
# Verify installation
git --version
```

**4. Flutter SDK (for mobile apps)**
```bash
# Download from: https://flutter.dev/docs/get-started/install

# Add to PATH
export PATH="$PATH:`pwd`/flutter/bin"

# Verify installation
flutter --version
flutter doctor  # Check for issues
```

**5. React Development (for admin dashboard)**
```bash
# Node.js already includes npm
# No additional installation needed
```

---

### Backend Setup

#### 1. Clone Repository
```bash
git clone https://github.com/parktayo/parktayo-system.git
cd parktayo-system/parktayo-backend
```

#### 2. Install Dependencies
```bash
npm install
```

#### 3. Environment Configuration
```bash
# Create .env file
cp .env.example .env

# Edit .env file with your configuration
nano .env  # or use your favorite editor
```

**.env Configuration:**
```bash
# Server
PORT=5000
NODE_ENV=development

# Database
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/parktayo?retryWrites=true&w=majority

# Authentication
JWT_SECRET=your-super-secret-jwt-key-change-this-in-production
JWT_EXPIRES_IN=7d

# Google Maps
GOOGLE_MAPS_API_KEY=AIzaSyC_your_api_key_here

# Firebase
FIREBASE_SERVICE_ACCOUNT=./config/firebase-service-account.json
FIREBASE_PROJECT_ID=parktayo-12345

# Cloudinary
CLOUDINARY_CLOUD_NAME=your-cloud-name
CLOUDINARY_API_KEY=123456789012345
CLOUDINARY_API_SECRET=your-api-secret
CLOUDINARY_URL=cloudinary://api_key:api_secret@cloud_name

# SMS (Semaphore - Philippines)
SEMAPHORE_API_KEY=your-semaphore-api-key

# Email (Optional)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password

# API Version
API_VERSION=v1

# Service Fee
PLATFORM_SERVICE_FEE_FLAT=5
PLATFORM_SERVICE_FEE_PERCENTAGE=5

# Booking Settings
MINIMUM_BOOKING_HOURS=3
GRACE_PERIOD_MINUTES=15
AUTO_REJECT_HOURS=4

# Pricing
OVERTIME_RATE_DEFAULT=15
MAX_DEMAND_FACTOR=0.5
```

#### 4. Firebase Setup
```bash
# Download Firebase service account key
# 1. Go to Firebase Console: https://console.firebase.google.com/
# 2. Select your project
# 3. Go to Project Settings → Service Accounts
# 4. Click "Generate New Private Key"
# 5. Save as: parktayo-backend/config/firebase-service-account.json
```

#### 5. Database Seed Data (Optional)
```bash
# Seed system settings
node src/scripts/seedSystemSettings.js

# Create super admin
node create-super-admin.js
```

#### 6. Start Development Server
```bash
# Start with nodemon (auto-reload)
npm run dev

# Or start normally
npm start
```

**Expected Output:**
```
🚀 Server is running on port 5000
✅ MongoDB connected successfully
✅ Firebase initialized
📡 API: http://localhost:5000/api/v1
```

#### 7. Test API
```bash
# Test health endpoint
curl http://localhost:5000/api/v1/health

# Expected response:
# {"status":"ok","timestamp":"2025-01-15T14:30:00.000Z"}
```

---

### User Mobile App Setup (parktayoflutter)

#### 1. Navigate to Project
```bash
cd ../parktayoflutter
```

#### 2. Install Dependencies
```bash
flutter pub get
```

#### 3. Environment Configuration

**Create .env file:**
```bash
# Copy example
cp .env.example .env
```

**.env Configuration:**
```
API_BASE_URL=http://10.0.2.2:5000/api/v1
GOOGLE_MAPS_API_KEY=AIzaSyC_your_api_key_here
ENVIRONMENT=development
```

**Note:** Use `10.0.2.2` for Android emulator, `localhost` for iOS simulator

#### 4. Firebase Configuration

**Android:**
```bash
# Download google-services.json from Firebase Console
# Place in: android/app/google-services.json
```

**iOS:**
```bash
# Download GoogleService-Info.plist from Firebase Console
# Place in: ios/Runner/GoogleService-Info.plist
```

#### 5. Run App

**Android:**
```bash
# List available devices
flutter devices

# Run on specific device
flutter run -d <device-id>

# Or run on default device
flutter run
```

**iOS:**
```bash
# Open iOS simulator
open -a Simulator

# Run app
flutter run
```

**Expected Output:**
```
✓ Built build/app/outputs/flutter-apk/app-debug.apk
Launching lib/main.dart on Android SDK built for x86 in debug mode...
Running Gradle task 'assembleDebug'...
✓ Built build/app/outputs/flutter-apk/app-debug.apk in 45.3s
Installing build/app/outputs/flutter-apk/app.apk...
Syncing files to device Android SDK built for x86...
Flutter run key commands.
h List all available interactive commands.
```

---

### Landlord Mobile App Setup (parktayo_landlord)

#### Steps (Similar to User App)
```bash
cd ../parktayo_landlord
flutter pub get

# Configure .env
cp .env.example .env

# Add Firebase config files
# android/app/google-services.json
# ios/Runner/GoogleService-Info.plist

# Run app
flutter run
```

---

### Admin Dashboard Setup (newparktayoadmin)

#### 1. Navigate to Project
```bash
cd ../newparktayoadmin
```

#### 2. Install Dependencies
```bash
npm install
```

#### 3. Environment Configuration

**Create .env file:**
```bash
# Copy example
cp .env.example .env
```

**.env Configuration:**
```
REACT_APP_API_BASE_URL=http://localhost:5000/api/v1
REACT_APP_ENVIRONMENT=development
REACT_APP_FIREBASE_API_KEY=your-firebase-api-key
REACT_APP_FIREBASE_AUTH_DOMAIN=parktayo-12345.firebaseapp.com
REACT_APP_FIREBASE_PROJECT_ID=parktayo-12345
REACT_APP_FIREBASE_STORAGE_BUCKET=parktayo-12345.appspot.com
REACT_APP_FIREBASE_MESSAGING_SENDER_ID=123456789012
REACT_APP_FIREBASE_APP_ID=1:123456789012:web:abcdef1234567890
```

#### 4. Start Development Server
```bash
npm start
```

**Expected Output:**
```
Compiled successfully!

You can now view parktayoadmin in the browser.

  Local:            http://localhost:3000
  On Your Network:  http://192.168.1.100:3000

Note that the development build is not optimized.
To create a production build, use npm run build.

webpack compiled successfully
```

#### 5. Access Dashboard
```
URL: http://localhost:3000
Default Login:
  Email: admin@parktayo.com
  Password: admin123
```

---

### Marketing Website Setup (parktayo-website)

#### 1. PHP Server
```bash
cd ../parktayo-website

# Option A: PHP Built-in Server
php -S localhost:8000

# Option B: XAMPP/WAMP/MAMP
# Place files in htdocs/www directory
```

#### 2. Access Website
```
URL: http://localhost:8000
```

---
