# Technology Stack

## Backend Technologies

#### Core Framework
```json
{
  "runtime": "Node.js 18+",
  "framework": "Express.js 4.x",
  "language": "JavaScript (ES6+)",
  "database": "MongoDB 6.0+",
  "odm": "Mongoose 8.x"
}
```

#### Key Dependencies
```json
{
  "authentication": "jsonwebtoken (JWT)",
  "security": "bcryptjs, helmet, express-rate-limit",
  "validation": "express-validator",
  "logging": "winston",
  "scheduling": "node-cron",
  "realtime": "socket.io",
  "email": "nodemailer",
  "sms": "twilio"
}
```

#### External Services
- **Google Maps API** - Routes, Places, Geocoding
- **Firebase** - Cloud Messaging (FCM), Authentication
- **Cloudinary** - Image upload and CDN
- **Semaphore** - SMS notifications (Philippines)
- **MongoDB Atlas** - Cloud database hosting

## Frontend Technologies

#### User Mobile App (parktayoflutter)
```yaml
Framework: Flutter 3.x
Language: Dart 3.x
State Management: Provider, setState
Key Packages:
  - google_maps_flutter: Maps integration
  - http: API communication
  - shared_preferences: Local storage
  - firebase_messaging: Push notifications
  - qr_flutter: QR code generation
  - mobile_scanner: QR code scanning
  - geolocator: Location services
```

#### Landlord Mobile App (parktayo_landlord)
```yaml
Framework: Flutter 3.x
Language: Dart 3.x
State Management: BLoC pattern, Provider
Key Packages:
  - firebase_core: Firebase integration
  - flutter_bloc: State management
  - freezed: Immutable models
  - qr_flutter: QR code generation
  - image_picker: Photo capture
```

#### Admin Dashboard (newparktayoadmin)
```yaml
Framework: React 19
Language: TypeScript 4.x
UI Library: Material-UI (MUI) 6.x
State Management: Redux Toolkit
Key Libraries:
  - react-router-dom: Routing
  - axios: HTTP client
  - recharts/chart.js: Data visualization
  - socket.io-client: Real-time updates
  - date-fns: Date manipulation
```

#### Marketing Website (parktayo-website)
```yaml
Platform: PHP
Frontend: HTML5, CSS3, JavaScript
Libraries:
  - Bootstrap: Responsive design
  - jQuery: DOM manipulation
  - Google Analytics: Tracking
```

## DevOps & Infrastructure

#### Development Tools
- **Version Control:** Git, GitHub
- **API Testing:** Postman, Thunder Client
- **Package Managers:** npm (Node.js), pub (Flutter)
- **Code Editors:** VS Code, Android Studio

#### Production Recommendations
```yaml
Hosting:
  API: AWS EC2 / Google Cloud / DigitalOcean
  Database: MongoDB Atlas
  CDN: Cloudflare
  SSL: Let's Encrypt

Monitoring:
  Application: PM2, New Relic
  Errors: Sentry
  Logs: Winston + Cloud Logging
  Performance: Google Analytics

Security:
  Firewall: AWS Security Groups
  DDoS Protection: Cloudflare
  Secrets: Environment Variables, AWS Secrets Manager
```

---
