# ParkTayo - Smart Parking Management System

A comprehensive smart parking management ecosystem built with modern technologies.

## Project Structure

This repository contains four main applications:

### 📱 **parktayoflutter** - Mobile App (Flutter)
The main mobile application for users to find and book parking spaces.
- **Technology**: Flutter/Dart
- **Features**: 
  - Real-time parking space availability
  - Smart booking system
  - Geofencing and location tracking
  - Multi-language support (English/Filipino)
  - Payment integration

### 🏢 **parktayo_landlord** - Landlord App (Flutter)
Mobile application for parking space owners and managers.
- **Technology**: Flutter/Dart
- **Features**:
  - Property management dashboard
  - Booking oversight
  - Revenue tracking
  - Real-time notifications

### 🌐 **newparktayoadmin** - Admin Web Dashboard (React)
Web-based administrative interface for system management.
- **Technology**: React/TypeScript
- **Features**:
  - System-wide analytics
  - User management
  - Content management
  - Configuration settings

### ⚡ **parktayo-backend** - Backend API (Node.js)
RESTful API server handling all backend operations.
- **Technology**: Node.js/Express
- **Features**:
  - User authentication and authorization
  - Booking management
  - Payment processing
  - Real-time notifications
  - Database management

## Getting Started

### Prerequisites
- Node.js (v16 or higher)
- Flutter SDK (v3.0 or higher)
- Git

### Installation

1. Clone the repository:
```bash
git clone https://github.com/gcash3/parktayo.git
cd parktayo
```

2. Set up each application:

#### Backend Setup
```bash
cd parktayo-backend
npm install
npm start
```

#### Admin Dashboard Setup
```bash
cd newparktayoadmin
npm install
npm start
```

#### Flutter Mobile App Setup
```bash
cd parktayoflutter
flutter pub get
flutter run
```

#### Flutter Landlord App Setup
```bash
cd parktayo_landlord
flutter pub get
flutter run
```

## Features

- 🚗 **Smart Parking Detection**: Real-time availability tracking
- 📍 **Geofencing**: Location-based services and restrictions
- 💳 **Integrated Payments**: Secure payment processing
- 🌐 **Multi-language Support**: English and Filipino localization
- 📊 **Analytics Dashboard**: Comprehensive reporting and insights
- 🔔 **Real-time Notifications**: Instant updates and alerts
- 🏛️ **LGU Compliance**: Government regulation compliance features

## Technology Stack

- **Mobile Apps**: Flutter, Dart
- **Web Dashboard**: React, TypeScript
- **Backend**: Node.js, Express
- **Database**: Firebase/Firestore
- **Authentication**: Firebase Auth
- **Maps**: Google Maps API
- **Payments**: Various payment gateways
- **Real-time**: WebSocket connections

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## License

This project is proprietary software. All rights reserved.

## Contact

For questions or support, please contact the development team.
