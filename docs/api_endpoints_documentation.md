# LinqUp API Documentation Structure

This document outlines the structured API requirements for the **LinqUp** platform. It has been categorized to facilitate easy generation of GitHub documentation. Shared functionalities (e.g., Auth, Profile Settings) are grouped separately to avoid duplication across Mobile App, Vendor, and Admin platforms.

---

## 1. Shared / Common Services
These endpoints are utilized across different user roles (Users, Vendors, Admins).

### **Authentication & Authorization**
- `POST /auth/google` - Google authentication
- `POST /auth/ios` - iOS Apple authentication
- `POST /auth/email` - Standard email/password authentication
- `POST /auth/password-reset` - Reset password
- `POST /auth/otp/dispatch` - Email OTP dispatch service

### **Account & Security Settings**
- `PUT /account/email` - Update email address
- `PUT /account/password` - Update password
- `POST /account/2fa/enable` - Enable 2-Factor Authentication (2FA)
- `POST /account/2fa/disable` - Disable 2FA
- `POST /account/feedback` - Send platform feedback
- `DELETE /account` - Delete account

### **Media & Utilities**
- `POST /media/upload` - Upload generic file (images, documents)

---

## 2. Mobile App (End Users)
Endpoints specific to the consumer-facing mobile application.

### **Onboarding & Profile Management**
- `GET /profile/interests` - Get available interests list
- `POST /profile/interests` - Add user interests
- `POST /profile/bio` - Add user bio
- `GET /profile/bio` - Get user bio
- `PUT /profile/photo` - Create/Update/Delete profile photo
- `POST /profile/face-verify` - Face verification
- `PUT /profile/location` - Set current location
- `GET /profile/sexual-orientations` - Get sexual orientations list
- `PUT /profile/sexual-orientation` - Set sexual orientation
- `PUT /profile/split` - Update split value for dates
- `POST /profile/notifications` - Enable/disable notifications

### **Discovery & Matching**
- `GET /recommendations` - Get recommended profiles
- `GET /recommendations/filter` - Filter recommendations
- `GET /profiles/:id` - Get specific profile details
- `POST /profiles/:id/like` - Like profile
- `POST /profiles/:id/reject` - Reject profile
- `POST /profiles/:id/share` - Share profile
- `GET /matches` - Get matches
- `GET /likes` - Get likes
- `POST /matches/:id/unmatch` - Un-match profile

### **Social Actions & Moderation**
- `POST /profiles/:id/block` - Block profile
- `POST /profiles/:id/report` - Report profile
- `GET /blocks` - Get blocked users
- `POST /blocks/:id/unblock` - Unblock user
- `POST /users/:id/nudge` - Nudge user

### **Restaurants & Date Planning**
- `GET /restaurants/proximity` - Get proximity restaurants
- `GET /restaurants/:id/menu` - Get restaurant menu
- `GET /restaurants/:id/reviews` - Get restaurant reviews
- `GET /restaurants/:id/reviews/summary` - Get review summary (sorted)
- `POST /dates/propose` - Propose locations
- `GET /dates` - View scheduled dates

### **Orders & Checkout**
- `POST /checkout` - Checkout / Payment for dates
- `GET /wallet/transactions` - Get all transactions
- `POST /wallet/fund` - Fund wallet

### **Chat & Communication**
- `GET /conversations` - Get conversations list
- `GET /conversations/:id/messages` - Get messages in a chat
- `POST /conversations/:id/messages` - Send text message
- `POST /conversations/:id/messages/voice` - Send voice message
- `POST /conversations/:id/messages/images` - Send images
- `POST /calls/voice` - Trigger voice call
- `POST /calls/video` - Trigger video call

### **Subscriptions**
- `GET /subscriptions` - Get available subscriptions plans
- `GET /subscriptions/features` - Get subscription features

---

## 3. Vendors (Restaurant Partners)
Endpoints for restaurant owners and managers on the Vendor web platform.

### **Dashboard & Analytics**
- `GET /vendor/dashboard` - Get dashboard overview (supports `start_date` & `end_date` query)
- `GET /vendor/analytics/revenue` - Total revenue in selected period & percentage increase
- `GET /vendor/analytics/status` - Status breakdown in percentages (confirmed | pending | partially confirmed | cancelled)

### **Restaurant Management**
- `POST /vendor/restaurants` - Add restaurant (Name, Address, Cover Photo, Logo, Manager Contact, Email, Phone)
- `PUT /vendor/restaurants/:id` - Edit restaurant details
- `GET /vendor/restaurants/:id/hours` - Get working hours
- `PUT /vendor/restaurants/:id/hours` - Add/update working hours
- `POST /vendor/menu` - Add menu item

### **Bookings & Orders**
- `GET /vendor/orders/recent` - Get recent orders
- `GET /vendor/orders` - View all orders
- `GET /vendor/orders/summary` - Get order summary (All | confirmed | pending | partially confirmed | cancelled)
- `GET /vendor/orders/:id/timeline` - Get order status timeline
- `POST /vendor/orders/verify` - Verify order code
- `GET /vendor/dates` - Get scheduled dates for restaurant
- `GET /vendor/dates/calendar` - Return dates for selected period to render in calendar
- `GET /vendor/dates/search` - Search dates

### **Finance & Payouts**
- `GET /vendor/finance/escrow` - Get escrow payments & records
- `GET /vendor/finance/deposits` - View deposits records
- `GET /vendor/finance/withdrawals` - View withdrawal records
- `GET /vendor/finance/refunds` - View refund records
- `POST /vendor/finance/withdraw` - Withdraw funds
- `POST /vendor/finance/payout-account` - Add payout account

### **Team Management**
- `GET /vendor/team` - Get team members
- `GET /vendor/team/export` - Export team members list
- `POST /vendor/team` - Add team member & assign role (admin | super_admin)
- `PUT /vendor/team/:id/status` - Activate/Deactivate team member
- `DELETE /vendor/team/:id` - Delete team member
- `GET /vendor/team/permissions` - Get roles & permissions list

### **Notifications**
- `GET /vendor/notifications` - Get vendor alerts & notifications

---

## 4. Admin (Platform Management)
Endpoints for the Super Admin web platform to oversee LinqUp completely.

### **Global Dashboard & Analytics**
- `GET /admin/activities` - Get recent platform activities
- `GET /admin/analytics` - Get detailed system analytics
- `GET /admin/revenue` - Get global revenue
- `GET /admin/todos` - Get admin to-dos/tasks

### **Users, Vendors & KYC**
- `GET /admin/users` - Get all users on the platform
- `GET /admin/vendors` - Get all vendors on the platform
- `GET /admin/kyc` - Get vendor/user KYC requests
- `POST /admin/kyc/:id/review` - Approve or decline KYC request
- `GET /admin/reports` - Get user/restaurant reports

### **Global Orders & Transactions**
- `GET /admin/orders` - Get global orders
- `GET /admin/orders/overview` - View platform-wide orders overview
- `GET /admin/orders/activities` - View order activities/history
- `GET /admin/users/:user_id/orders` - View specific customer orders
- `GET /admin/transactions` - Get global financial transactions
- `GET /admin/schedules` - Get global scheduled dates (linqups)
- `GET /admin/schedules/calendar` - Get global schedules grouped for calendar view

### **Subscriptions & Audits**
- `GET /admin/subscriptions/recent` - Get recent subscription payments
- `GET /admin/subscriptions` - Manage global subscriptions
- `GET /admin/audit-logs` - View platform audit logs
- `GET /admin/notifications` - Platform notifications & system alerts

### **Admin Team Management**
- `GET /admin/team` - Manage internal admin team
- `POST /admin/team` - Add admin user
