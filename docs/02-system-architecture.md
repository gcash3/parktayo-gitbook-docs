# System Architecture

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        FRONTEND LAYER                        │
├──────────────┬──────────────┬──────────────┬────────────────┤
│   Mobile     │   Mobile     │     Web      │      Web       │
│   (User)     │  (Landlord)  │   (Admin)    │  (Marketing)   │
│   Flutter    │   Flutter    │    React     │      PHP       │
└──────┬───────┴──────┬───────┴──────┬───────┴────────┬───────┘
       │              │              │                │
       └──────────────┴──────────────┴────────────────┘
                            │
                            ▼
       ┌─────────────────────────────────────────────┐
       │            API GATEWAY / LOAD BALANCER       │
       └─────────────────────────────────────────────┘
                            │
                            ▼
       ┌─────────────────────────────────────────────┐
       │         BACKEND API (Node.js/Express)       │
       │  ├── Authentication & Authorization          │
       │  ├── Business Logic & Controllers            │
       │  ├── AI Services & Dynamic Pricing           │
       │  ├── Booking Lifecycle Management            │
       │  └── Real-time Notifications                 │
       └─────────────────────────────────────────────┘
                            │
          ┌─────────────────┴─────────────────┐
          ▼                                   ▼
    ┌──────────────┐                  ┌──────────────┐
    │   MongoDB    │                  │   Firebase   │
    │  (Database)  │                  │ (Auth/Push)  │
    └──────────────┘                  └──────────────┘
          │                                   │
          ▼                                   ▼
    ┌──────────────┐                  ┌──────────────┐
    │   Cloudinary │                  │ Google Maps  │
    │   (Images)   │                  │  Routes API  │
    └──────────────┘                  └──────────────┘
```

## Data Flow Architecture

#### Booking Flow:
```
User → Mobile App → Backend API → Check Availability →
Calculate Pricing → Hold Wallet Amount → Create Booking →
Schedule No-Show Check → Notify Landlord → Return Confirmation
```

#### Payment Flow:
```
User → Top-up Wallet → Escrow Service → Book Parking →
Amount Held → User Checks Out → QR Scan/Manual →
Capture Payment → Split Revenue (Landlord + Platform) →
Release to Wallets
```

#### Smart Booking Flow:
```
User → Current Location → Destination Selection →
Google Routes API (ETA) → Add Grace Period →
Create Time Window → Arrival Monitoring →
Geolocation Tracking → Arrival Detection → Start Parking Session
```

## Microservices Architecture

| Service | Purpose | Technology |
|---------|---------|------------|
| **Auth Service** | User authentication, JWT management | Node.js, JWT |
| **Booking Service** | Booking CRUD, availability checking | Node.js, MongoDB |
| **Smart Booking Service** | ETA calculation, arrival prediction | Node.js, Google Maps API |
| **Pricing Service** | Dynamic pricing, demand factors | Node.js, Custom Algorithm |
| **Wallet Service** | Payment processing, escrow | Node.js, MongoDB |
| **Notification Service** | Push notifications, SMS | Firebase FCM, Semaphore API |
| **AI Service** | Recommendations, analytics | Node.js, ML algorithms |
| **Image Service** | Photo upload, processing | Cloudinary |
| **Location Service** | Geospatial queries, tracking | MongoDB GeoJSON |

---
