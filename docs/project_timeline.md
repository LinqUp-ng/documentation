# LinqUp Project Master Timelines

**Methodology:** UI-First Development.
We will build the **Mobile Screens** first to validate the UX/Flow, and then design the **Database & Backend** immediately after to support those specific screens.

---

## Option A: Realistic Timeline (14 Weeks)

### Phase 1: Core UI & Identity (Weeks 1-3)

_Goal: Look and feel is complete. User can navigate the app._

- [ ] **Mobile Project Setup**
    - Navigation (Tabs, Stacks), Theming, Global State.
- [ ] **Auth & Profile Screens**
    - Build Login, OTP input, Registration, Onboarding Swiper.
    - Build User Profile, Edit Profile, Settings screens.
- [ ] **Backend Support (Follow-up)**
    - Setup Supabase & Email Service (Auth).
    - Design `users` schema & RLS policies to match the UI fields.

### Phase 2: Discovery & Matching (Weeks 4-6)

_Goal: The core "Dating" loop._

- [ ] **Matching UI**
    - Finish `SwipeableStack` interactions.
    - Build "It's a Match!" modal/screen.
- [ ] **Backend Support**
    - Design `swipes` and `matches` schema.
    - write "Matching Logic" Edge Function.

### Phase 3: Communication & Media (Weeks 7-8)

_Goal: Social interaction._

- [ ] **Chat & Call UI**
    - Build Chat Room List & Chat Detail (Bubble UI).
    - Build Video Call Interface (Overlay, Controls).
- [ ] **Backend Support**
    - Setup Realtime channels.
    - Integrate WebRTC Signaling (Agora/Stream).

### Phase 4: The Ecosystem (Vendors) (Weeks 9-10)

_Goal: Connecting with the physical world._

- [ ] **Hangout UI**
    - Build Vendor Discovery (Map/List).
    - Build "Reservation/Booking" modal.
- [ ] **Backend Support**
    - Design `vendors` and `bookings` schema.
    - Create API for Vendor/Admin Web Apps.

### Phase 5: Finance (Weeks 11-13)

_Goal: Money handling._

- [ ] **Wallet UI**
    - Build Wallet Dashboard, Transaction History, Top-up Screen.
    - Build "Premium/Boost" Purchase Screens.
- [ ] **Backend Support**
    - Design `ledger` schema (Critical).
    - Integrate **Monnify** (Fiat -> Wallet).
    - Integrate **RevenueCat** (IAP).

### Phase 6: Polish & Launch (Week 14)

- [ ] Deep Linking & Push Notifications.
- [ ] Final QA & App Store Submission.

---

## Option B: Aggressive Timeline (12 Weeks)

### Month 1: The "App Structure" Blitz (Weeks 1-4)

_Goal: A clickable, navigable app with functioning Auth & Chat._

- [ ] **Week 1: Identity UI & Auth**
    - **Frontend:** Build Nav, Auth Screens, Profile Edit, Settings.
    - **Backend:** Setup Supabase, Auth, Email Service, `users` table.
- [ ] **Week 2: The Swipe Deck**
    - **Frontend:** `SwipeableStack`, Match Animations, filters.
    - **Backend:** `matches` schema & Edge Function.
- [ ] **Week 3: Chat & Video UI**
    - **Frontend:** Chat Interface & Video Call Overlay.
    - **Backend:** Realtime setup & WebRTC integrations.
- [ ] **Week 4: Discovery UI**
    - **Frontend:** Map View for Vendors, Vendor Profile Screen.
    - **Backend:** `vendors` schema initial draft.

### Month 2: The "Economy" Sprint (Weeks 5-8)

_Goal: Heavy logic integration (Monnify, RevenueCat)._

- [ ] **Week 5: Wallet UI & Logic**
    - **Frontend:** Wallet Screens.
    - **Backend:** Ledger System implementation.
- [ ] **Week 6: Payments Integration**
    - **Shared:** Integrate **Monnify** SDK & Webhooks.
- [ ] **Week 7: IAP & Premium Features**
    - **Frontend:** Paywalls & "Buy Boost" UI.
    - **Backend:** **RevenueCat** Wiring.
- [ ] **Week 8: Booking Loop**
    - **Frontend:** Reservation UI.
    - **Backend:** Logic to lock funds in Wallet during booking.

### Month 3: Admin, Polish & Ship (Weeks 9-12)

_Goal: Web support and finishing touches._

- [ ] **Week 9: Admin/Vendor APIs**
    - **Backend:** Expose endpoints for the Web Apps.
- [ ] **Week 10: Notifications & Deep Links**
    - **System:** Configure triggers for "New Match", "Money Received".
- [ ] **Week 11: QA & Security**
    - **Audit:** Check RLS policies against UI actions.
- [ ] **Week 12: Launch**
    - **Ship:** Store release.
