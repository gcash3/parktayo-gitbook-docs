# SECTION 6: TROUBLESHOOTING GUIDE

## 6.1 Common Backend Issues

### Issue: MongoDB Connection Failed

**Error Message:**
```
MongoServerError: Authentication failed.
```

**Causes:**
1. Incorrect MongoDB URI
2. Wrong username/password
3. IP not whitelisted (MongoDB Atlas)
4. Network connectivity issues

**Solutions:**

**Solution 1: Check MongoDB URI**
```bash
# Verify .env file
cat .env | grep MONGODB_URI

# Test connection
mongosh "mongodb+srv://your-uri"
```

**Solution 2: Whitelist IP (MongoDB Atlas)**
```
1. Go to MongoDB Atlas Dashboard
2. Network Access → Add IP Address
3. Add current IP or 0.0.0.0/0 (for testing only!)
```

**Solution 3: Check Credentials**
```
1. MongoDB Atlas → Database Access
2. Verify username exists
3. Reset password if needed
4. Update .env with new credentials
```

---

### Issue: JWT Token Expired

**Error Message:**
```json
{
  "status": "error",
  "message": "Token expired"
}
```

**Causes:**
1. Token expired (default 7 days)
2. System time incorrect
3. Token blacklisted

**Solutions:**

**Solution 1: Refresh Token**
```javascript
// Frontend - Auto refresh before expiry
const checkTokenExpiry = () => {
  const token = localStorage.getItem('token');
  const decoded = jwt_decode(token);
  const expiryTime = decoded.exp * 1000;
  const currentTime = Date.now();

  if (expiryTime - currentTime < 24 * 60 * 60 * 1000) {
    // Refresh if less than 24 hours remaining
    refreshToken();
  }
};
```

**Solution 2: Update JWT Expiry**
```bash
# In .env
JWT_EXPIRES_IN=30d  # Increase to 30 days
```

---

### Issue: Port Already in Use

**Error Message:**
```
Error: listen EADDRINUSE: address already in use :::5000
```

**Solutions:**

**Solution 1: Kill Process (Windows)**
```cmd
netstat -ano | findstr :5000
taskkill /PID <PID> /F
```

**Solution 2: Kill Process (Linux/Mac)**
```bash
lsof -ti:5000 | xargs kill -9
```

**Solution 3: Use Different Port**
```bash
# In .env
PORT=5001
```

---

### Issue: CORS Error

**Error Message:**
```
Access to fetch at 'http://localhost:5000/api/v1/auth/login' from origin 'http://localhost:3000'
has been blocked by CORS policy
```

**Solutions:**

**Solution: Configure CORS**
```javascript
// src/server.js
const cors = require('cors');

app.use(cors({
  origin: [
    'http://localhost:3000',
    'http://localhost:8080',
    'https://admin.parktayo.com',
    'https://www.parktayo.com'
  ],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

---

### Issue: File Upload Fails

**Error Message:**
```
MulterError: File too large
```

**Solutions:**

**Solution 1: Increase File Size Limit**
```javascript
// src/middleware/upload.js
const multer = require('multer');

const upload = multer({
  storage: cloudinaryStorage,
  limits: {
    fileSize: 10 * 1024 * 1024 // 10 MB
  }
});
```

**Solution 2: Check Cloudinary Configuration**
```bash
# Verify .env
CLOUDINARY_CLOUD_NAME=your-cloud-name
CLOUDINARY_API_KEY=your-api-key
CLOUDINARY_API_SECRET=your-api-secret
```

---

## 6.2 Common Frontend Issues

### Issue: Flutter Build Failed (Android)

**Error Message:**
```
FAILURE: Build failed with an exception.
```

**Solutions:**

**Solution 1: Clean Build**
```bash
flutter clean
flutter pub get
flutter build apk
```

**Solution 2: Update Gradle**
```gradle
// android/build.gradle
dependencies {
    classpath 'com.android.tools.build:gradle:7.3.0'
}
```

**Solution 3: Check Java Version**
```bash
java -version  # Should be Java 11 or later

# Set JAVA_HOME
export JAVA_HOME=/path/to/java11
```

---

### Issue: Google Maps Not Showing

**Error Message:**
```
PlatformException(INVALID_API_KEY)
```

**Solutions:**

**Solution 1: Enable APIs**
```
1. Go to Google Cloud Console
2. Enable APIs:
   - Maps SDK for Android
   - Maps SDK for iOS
   - Places API
   - Routes API
3. Wait 5-10 minutes for propagation
```

**Solution 2: Check API Key Configuration**

**Android:**
```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<meta-data
    android:name="com.google.android.geo.API_KEY"
    android:value="YOUR_API_KEY_HERE"/>
```

**iOS:**
```xml
<!-- ios/Runner/AppDelegate.swift -->
GMSServices.provideAPIKey("YOUR_API_KEY_HERE")
```

---

### Issue: Firebase Push Notifications Not Working

**Causes:**
1. FCM token not generated
2. Firebase configuration incorrect
3. Notifications permissions not granted

**Solutions:**

**Solution 1: Check Firebase Configuration**
```bash
# Verify files exist
ls android/app/google-services.json
ls ios/Runner/GoogleService-Info.plist
```

**Solution 2: Request Permissions**
```dart
// Request notification permissions
FirebaseMessaging messaging = FirebaseMessaging.instance;

NotificationSettings settings = await messaging.requestPermission(
  alert: true,
  badge: true,
  sound: true,
);

if (settings.authorizationStatus == AuthorizationStatus.authorized) {
  print('User granted permission');
}
```

**Solution 3: Generate FCM Token**
```dart
String? token = await FirebaseMessaging.instance.getToken();
print('FCM Token: $token');

// Send token to backend
await apiService.updateFCMToken(token);
```

---

### Issue: API Calls Failing in Flutter

**Error Message:**
```
SocketException: Failed host lookup: 'api.parktayo.com'
```

**Solutions:**

**Solution 1: Check Network Connectivity**
```dart
import 'package:connectivity_plus/connectivity_plus.dart';

final connectivityResult = await Connectivity().checkConnectivity();
if (connectivityResult == ConnectivityResult.none) {
  // Show no internet message
}
```

**Solution 2: Use Correct URL for Emulator**
```dart
// For Android Emulator, use 10.0.2.2
final String baseUrl = Platform.isAndroid
    ? 'http://10.0.2.2:5000/api/v1'
    : 'http://localhost:5000/api/v1';
```

**Solution 3: Add Internet Permission**
```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<uses-permission android:name="android.permission.INTERNET"/>
```

---

## 6.3 Common Admin Dashboard Issues

### Issue: React Build Fails

**Error Message:**
```
Module not found: Can't resolve '@mui/material'
```

**Solutions:**

**Solution 1: Reinstall Dependencies**
```bash
rm -rf node_modules package-lock.json
npm install
```

**Solution 2: Clear Cache**
```bash
npm cache clean --force
rm -rf node_modules/.cache
npm start
```

---

### Issue: Admin Cannot Login

**Causes:**
1. Admin user not created
2. Wrong admin level
3. Password incorrect

**Solutions:**

**Solution 1: Create Admin User**
```bash
cd parktayo-backend
node create-super-admin.js
```

**Solution 2: Reset Admin Password**
```bash
node reset-admin-password.js admin@parktayo.com newpassword123
```

**Solution 3: Verify Admin Level**
```javascript
// Check in MongoDB
db.admins.findOne({ email: 'admin@parktayo.com' })

// Update admin level
db.admins.updateOne(
  { email: 'admin@parktayo.com' },
  { $set: { adminLevel: 'super_admin' } }
)
```

---

## 6.4 Database Issues

### Issue: Slow Query Performance

**Symptoms:**
- API responses taking > 1 second
- High database CPU usage
- Timeouts

**Solutions:**

**Solution 1: Add Indexes**
```javascript
// Check current indexes
db.bookings.getIndexes()

// Add compound index for booking queries
db.bookings.createIndex({
  parkingSpaceId: 1,
  startTime: 1,
  endTime: 1,
  status: 1
})

// Add geospatial index
db.parkingspaces.createIndex({ location: "2dsphere" })
```

**Solution 2: Optimize Queries**
```javascript
// Bad - Fetches all fields
const bookings = await Booking.find({ userId });

// Good - Select only needed fields
const bookings = await Booking.find({ userId })
  .select('startTime endTime status pricing')
  .lean();
```

**Solution 3: Use Aggregation Pipeline**
```javascript
// Optimize complex queries
const stats = await Booking.aggregate([
  { $match: { userId: mongoose.Types.ObjectId(userId) } },
  { $group: {
      _id: '$status',
      count: { $sum: 1 },
      totalAmount: { $sum: '$pricing.totalAmount' }
    }
  }
]);
```

---

### Issue: MongoDB Atlas Connection Timeout

**Error Message:**
```
MongoServerSelectionError: connection timed out
```

**Solutions:**

**Solution 1: Check Network Access**
```
1. MongoDB Atlas → Network Access
2. Add IP address
3. Or add 0.0.0.0/0 for development (not recommended for production)
```

**Solution 2: Check Connection String**
```bash
# Verify format
mongodb+srv://username:password@cluster.mongodb.net/database?retryWrites=true&w=majority

# Test connection
mongosh "your-connection-string"
```

**Solution 3: Increase Timeout**
```javascript
mongoose.connect(process.env.MONGODB_URI, {
  serverSelectionTimeoutMS: 30000, // 30 seconds
  socketTimeoutMS: 45000
});
```

---

## 6.5 Deployment Issues

### Issue: PM2 Process Crashes

**Symptoms:**
- API not responding
- PM2 shows status "errored"

**Solutions:**

**Solution 1: Check Logs**
```bash
pm2 logs parktayo-api --lines 100
pm2 logs parktayo-api --err  # Error logs only
```

**Solution 2: Increase Memory Limit**
```bash
pm2 start src/server.js --name parktayo-api --max-memory-restart 500M
```

**Solution 3: Auto-Restart on Crash**
```bash
pm2 start src/server.js --name parktayo-api --exp-backoff-restart-delay=100
```

---

### Issue: SSL Certificate Error

**Error Message:**
```
NET::ERR_CERT_AUTHORITY_INVALID
```

**Solutions:**

**Solution 1: Renew Certificate**
```bash
sudo certbot renew
sudo systemctl reload nginx
```

**Solution 2: Force HTTPS Redirect**
```nginx
server {
    listen 80;
    server_name api.parktayo.com;
    return 301 https://$server_name$request_uri;
}
```

---

## 6.6 Performance Optimization

### Backend Optimization Checklist

- [ ] Enable compression
- [ ] Add database indexes
- [ ] Implement caching (Redis)
- [ ] Optimize database queries
- [ ] Use connection pooling
- [ ] Enable GZIP
- [ ] Minify responses
- [ ] Use CDN for static assets
- [ ] Implement rate limiting
- [ ] Monitor with APM tools

**Enable Compression:**
```javascript
const compression = require('compression');
app.use(compression());
```

**Implement Redis Caching:**
```javascript
const redis = require('redis');
const client = redis.createClient({
  host: 'localhost',
  port: 6379
});

// Cache parking spaces
const getCachedParkingSpaces = async (key) => {
  const cached = await client.get(key);
  if (cached) {
    return JSON.parse(cached);
  }

  const spaces = await ParkingSpace.find().lean();
  await client.setex(key, 300, JSON.stringify(spaces)); // 5 min cache
  return spaces;
};
```

---

### Frontend Optimization

**Flutter Performance:**
```dart
// Use const constructors
const Text('Hello');

// Implement list virtualization
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) => ItemWidget(items[index]),
);

// Cache network images
CachedNetworkImage(
  imageUrl: imageUrl,
  placeholder: (context, url) => CircularProgressIndicator(),
);
```

**React Performance:**
```javascript
// Use React.memo
const ParkingCard = React.memo(({ parking }) => {
  return <div>{parking.name}</div>;
});

// Use useMemo for expensive calculations
const sortedSpaces = useMemo(
  () => spaces.sort((a, b) => a.price - b.price),
  [spaces]
);

// Lazy load components
const Dashboard = lazy(() => import('./pages/Dashboard'));
```

---

## 6.7 Debugging Tools

### Backend Debugging

**Winston Logger Setup:**
```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

// Use logger
logger.info('Booking created', { bookingId, userId });
logger.error('Database error', { error: err.message });
```

**Debug Mode:**
```bash
# Run with debug output
DEBUG=* npm start

# Debug specific namespace
DEBUG=express:* npm start
```

---

### Frontend Debugging

**Flutter DevTools:**
```bash
flutter pub global activate devtools
flutter pub global run devtools
```

**React DevTools:**
```
Install React DevTools browser extension
Open browser dev tools → React tab
```

---

## 6.8 Support Resources

### Getting Help

**Documentation:**
- Full documentation: `GITBOOK_MASTER_INDEX.md`
- API Reference: `GITBOOK_DEPLOYMENT_TESTING_TROUBLESHOOTING.md` → Section 3
- User Guide: `GITBOOK_COMPLETE_GUIDE.md` → Section 1

**Contact Support:**
- Email: support@parktayo.com
- Phone: 0942 463 8843
- Address: 2219 Recto Ave, Sampaloc, Manila, Philippines

**Community:**
- GitHub Issues: Report bugs
- GitHub Discussions: Ask questions
- Stack Overflow: Tag [parktayo]

**Emergency:**
- Critical production issues: emergency@parktayo.com
- Response time: < 1 hour

---

## 6.9 Maintenance Checklist

### Daily Tasks
- [ ] Check server health
- [ ] Monitor error logs
- [ ] Review support tickets
- [ ] Check database performance

### Weekly Tasks
- [ ] Review system metrics
- [ ] Update dependencies
- [ ] Run security scans
- [ ] Backup verification

### Monthly Tasks
- [ ] Performance audit
- [ ] Security review
- [ ] Update documentation
- [ ] Review and optimize queries

---
