# LinqUp Backend Development Roadmap

This document outlines the step-by-step plan for implementing the endpoints, tables, and functions required for the LinqUp platform, following the user journey and relational dependencies.

---

## 🚀 Phase 0: System Core (The Foundation)

_Establishing global configuration, security, and authentication._

### 0. Infrastructure

- [x] **Supabase Initialization:** Database setup, auth config, and storage buckets.

### 1. Global Platform Settings

-
- [ ] [X] **Table:** `public.global_settings` (earning %, min withdrawal, maintenance mode).
- [ ] **Endpoint:** `GET /admin/settings` - Retrieve all system configurations.
  - **Handler:** `api/handleGlobalSettings.ts` -> `getAllSettings()`
- [ ] **Endpoint:** `PUT /admin/settings/:key` - Update a specific global fee or toggle.
  - **Handler:** `api/handleGlobalSettings.ts` -> `updateSettingByKey()`
- [ ] **Endpoint:** `POST /admin/settings` - Create a new system setting.
  - **Handler:** `api/handleGlobalSettings.ts` -> `createNewSetting()`

### 2. Unified Security & Auth

- [ ] **Auth Providers:** `POST /auth/google`, `POST /auth/ios`, `POST /auth/email`.
- [ ] **OTP Service:** `POST /auth/otp/dispatch` - Generic service for registration, 2FA, and password reset.
- [ ] **Account Security:** `PUT /account/password`, `POST /account/2fa/enable`, `POST /account/2fa/disable`.
- [ ] **Logic:** Implement JWT Claims sync for `type` (admin/vendor/user) and `is_deactivated`.

---

## 👮 Phase 1: Admin & Vendor Management

_Setting up the ecosystem and internal team controls._

### 1. Admin Operations

- [ ] **Dashboard:** `GET /admin/activities`, `GET /admin/analytics`, `GET /admin/revenue`.
- [ ] **KYC & Moderation:** `GET /admin/kyc`, `POST /admin/kyc/:id/review`, `GET /admin/reports`.
- [ ] **Team:** `GET /admin/team`, `POST /admin/team` (Add with system-generated password).

### 2. Vendor Onboarding & Teams

- [ ] **Onboarding:** `POST /admin/vendors` (Invite business), `GET /admin/vendors`.
- [ ] **Vendor Team:** `GET /vendor/team`, `POST /vendor/team` (Invite staff with assigned roles).
- [ ] **Permissions:** `GET /vendor/team/permissions` - Fetch role-based access control (RBAC).

---

## 🍳 Phase 2: Store Content (The "Content" Layer)

_Vendors must prepare their stores and menus before user interaction._

### 1. Store Identity

- [ ] **Identity:** `PUT /vendor/stores/:id` - Update logo, cover photo, and address.
- [ ] **Operations:** `GET /vendor/stores/:id/hours`, `PUT /vendor/stores/:id/hours` (Working days).
- [ ] **Finance:** `POST /vendor/finance/payout-account` - Link Paystack-compatible account.

### 2. Menu Management

- [ ] **Categorization:** `public.food_categories` - Starters, Mains, Drinks.
- [ ] **Catalog:** `POST /vendor/menu`, `PUT /vendor/menu/:id` - Item with price, stock, and descriptions.
- [ ] **Options/Customization:** `POST /vendor/menu/options` - Required sides/extras.

---

## 💑 Phase 3: User Discovery & Connectivity

_Allowing customers to find each other and establish matches._

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

---

## 📅 Phase 4: The Booking Journey (Transactional)

_Date planning, payments, and booking redemption._

### 1. Date Planning

- [ ] **Proposals:** `GET /stores/proximity`, `POST /dates/propose`, `POST /dates/respond`.
- [ ] **Reminders:** 24-hour automated notification before `match.date_time`.

### 2. Wallet & Checkout

- [ ] **Wallet:** `GET /wallet/transactions`, `POST /wallet/fund`.
- [ ] **Payment:** `POST /checkout`, `POST /payment/pay` - Supports split billing or full payment.
- [ ] **Commission:** Automated platform fee deduction via `public.process_order_payment`.

### 3. Redemption

- [ ] **Verification:** `POST /vendor/orders/verify` - Scanner integration for booking code redemption.
- [ ] **Escrow Release:** Auto-transfer from platform escrow wallet to vendor's operating wallet.

---

## 📊 Phase 5: Analytics & Operations

_Post-date reviews and platform auditing._

### 1. Feedback & Reviews

- [ ] **Perks:** `GET /stores/:id/reviews`, `POST /stores/:id/reviews` (Rating + Perk selection).
- [ ] **Support:** `POST /account/feedback` - Direct platform communication.

### 2. Admin Financial Audits

- [ ] **Audits:** `GET /admin/transactions`, `GET /admin/subscriptions`, `GET /admin/audit-logs`.
- [ ] **Withdrawals:** `POST /vendor/finance/withdraw`, `GET /vendor/finance/withdrawals`.
