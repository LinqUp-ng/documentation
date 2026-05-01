# Phase 3: User Discovery & Connectivity
*Allowing customers to find each other and establish matches.*

### 1. Profile & Onboarding
- [ ] **Experience:** `GET /profile/interests`, `POST /profile/interests`, `PUT /profile/bio`.
- [ ] **Attributes:** `PUT /profile/photo`, `POST /profile/face-verify`, `PUT /profile/sexual-orientation`.
- [ ] **Growth:** `POST /profile/onboarding` - Handle referral logic.

### 2. Discovery Engine
- [ ] **Matching:** `GET /recommendations`, `POST /profiles/:id/like`, `POST /profiles/:id/reject`.
- [ ] **Social Filters:** `GET /recommendations/filter`, `GET /likes`, `GET /matches`.
- [ ] **Safety:** `POST /profiles/:id/block`, `POST /profiles/:id/report`.

### 3. Messaging
- [ ] **History:** `GET /conversations`, `GET /conversations/:id/messages`.
- [ ] **Real-time:** `POST /conversations/:id/messages` (Text, voice notes, and images).
- [ ] **Calls:** `POST /calls/voice`, `POST /calls/video` (ZegoCloud integration).
