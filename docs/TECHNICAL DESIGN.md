# Campus Driver App — Technical Design Document

**Version:** 1.1  
**Date:** February 21, 2026  
**Platform:** Mobile (iOS & Android)  
**Frontend:** Flutter  
**Backend:** Node.js + Express  
**Database:** PostgreSQL  
**Hosting:** AWS  

---

# 1. System Architecture

## 1.1 Overview

The Campus Driver App follows a **client–server architecture** with:

- Flutter mobile frontend  
- Node.js / Express REST API backend  
- PostgreSQL persistent storage  
- Firebase authentication & push notifications  
- Socket.io real-time communication  

---

## 1.2 Tech Stack Summary

| Layer | Technology |
|------|-----------|
| Mobile App | Flutter (iOS 14+ / Android 8.0+) |
| Backend | Node.js + Express |
| Database | PostgreSQL (AWS RDS) |
| Authentication | Firebase Auth |
| Push Notifications | Firebase Cloud Messaging (FCM) |
| Real-time Chat | Socket.io |
| Location / Maps | Photon Komoot API |
| Hosting | AWS EC2 (primary application hosting) |
| File Storage | AWS S3 (user uploads and assets) |

---

# 2. Database Schema

## 2.1 Entity Relationship Overview

```
users ───── rides
│           │
├── driver_profiles   ├── messages
└── fcm_tokens        └── notifications
```

---

## 2.2 Table Definitions (Key Columns)

### users
Stores PII and university verification status.

### driver_profiles
Stores vehicle details and `verification_status` (`pending`, `approved`).

### rides
Core table tracking:

- pickup/dropoff coordinates  
- timestamps  
- status (`pending`, `accepted`, `ongoing`, `completed`)  

### messages
Chat history with a `deleted_at` column for the daily cleanup job.

---

# 3. API Design & Environment Config

## 3.1 Centralized Domain Configuration

To avoid **config debt**, the system uses environment variables to define the domain in a single place.

**Backend (.env)**

```
APP_DOMAIN=campus-driver-app.xmum-codingclub.com
```

**Frontend (dart-define)**

```
API_BASE_URL=campus-driver-app.xmum-codingclub.com
```

**Base URL Construction**

```
REST: https://${API_BASE_URL}/v1
WebSockets: wss://${API_BASE_URL}
```

---

## 3.2 Authentication Endpoints

```
POST /auth/register
```
Links Firebase UID to a local PostgreSQL user.

```
POST /auth/fcm-token
```
Updates the device token for push notifications.

---

## 3.3 Ride & Driver Endpoints

```
GET /rides/available
```
Drivers only — filtered list of pending requests.

```
PATCH /rides/:id/accept
```
Uses PostgreSQL row-level locking (`FOR UPDATE`) to prevent multiple drivers accepting the same ride.

```
GET /location/search
```
Backend proxy to Photon Komoot API to bypass CORS and hide provider logic.

---

# 4. Real-time Communication (Socket.io)

## 4.1 Connection & Security

1. Flutter client connects using Firebase ID Token  
2. Server verifies token via Firebase Admin SDK during handshake  

---

## 4.2 Chat Rooms

Each ride has a unique room:

```
ride_{ride_id}
```

**Access Rule**

Only assigned:

- passenger_id  
- driver_id  

may join.

**Retention Policy**

Daily cron job at **midnight (UTC+8)** marks messages as deleted for all non-active rides.

---

## 4.3 Chat & Notification Flow (High-Level)

### 4.3.1 Chat Message Flow

1. User opens the ride chat screen in the Flutter app.  
2. Flutter connects to Socket.io using the Firebase ID token and joins `ride_{ride_id}`.  
3. When the passenger or driver sends a message, the client emits a `message:send` event containing:
   - `ride_id`  
   - sender `user_id`  
   - message body and metadata (timestamp, optional attachments)  
4. The backend:
   - verifies the Firebase ID token and that the user is part of the ride (`passenger_id` or `driver_id`)  
   - persists the message into the `messages` table  
   - emits a `message:new` event to the `ride_{ride_id}` room so all connected clients update in real time  
5. If the other participant is not actively connected to the room, the backend will trigger a push notification (via FCM) to that user’s registered devices (from `fcm_tokens`).  

### 4.3.2 Push Notifications (FCM)

- **Chat notifications**
  - Triggered when a new message is stored for a ride and the recipient is not in the active chat.  
  - Backend sends an FCM data notification with:
    - `ride_id`  
    - sender name  
    - short preview of the message  
- **Ride status notifications**
  - When key ride events occur (created, accepted, cancelled, completed), the backend:
    - looks up the relevant user(s) in `fcm_tokens`  
    - sends an FCM notification with a short, human-readable summary (e.g. “Your driver has accepted your ride”).  
  - The Flutter app deep-links the user into the appropriate screen (ride details or chat) based on the payload.  

---

# 5. Security & Auth Flow

## 5.1 Authentication Flow

1. Flutter app authenticates with Firebase  
2. Firebase returns JWT (ID Token)  
3. Flutter sends token in header  

```
Authorization: Bearer <firebase_id_token>
```

4. Node.js verifies token  
5. If valid → attaches `user_id` to request context  

---

## 5.2 Driver Verification

Access granted via **signed URLs** generated by backend for admin review.

---

# 6. Background Jobs

| Job | Schedule | Action |
|-----|---------|--------|
| Ride Expiration | Every 5 mins | Cancels pending rides older than 2 hours |
| Auto-Complete | Every 5 mins | Sets ongoing rides to completed after 3 hours |
| Message Cleanup | Daily 00:00 | Soft-deletes chat history for closed rides |

---

# 7. Flutter App Architecture

## 7.1 State Management

Riverpod providers:

- `authProvider` — token refresh & user profile  
- `rideStreamProvider` — real-time ride status via Socket.io  

---

## 7.2 UI Requirements

**Chat View**

Display persistent muted label:

```
ⓘ Messages will be cleared at end of day.
```

**Offline Mode**

Use `flutter_secure_storage` to cache last known user profile for faster startup.

---

# 8. Error Handling

Standard HTTP status codes:

```
409 Conflict
```
Driver attempted to accept already-taken ride.

```
410 Gone
```
Passenger interacted with expired ride.

---

# 10. Environment Configuration

All domain-specific strings must use environment variables for portability.

---

## 10.1 Backend Environment (.env)

Central configuration object variable:

```
APP_DOMAIN
```

**Usage**

- CORS policy  
- Absolute asset URLs  
- Internal token issuer  
- Socket.io allowed origins  

---

## 10.2 Flutter Configuration (dart-define)

Base URL variable:

```
API_BASE_URL
```

**Mapping**

```
REST: https://${API_BASE_URL}/v1
WebSockets: wss://${API_BASE_URL}
```

---

# End of Document