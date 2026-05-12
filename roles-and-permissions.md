# Roles and Permissions

This document defines the roles and permissions for Admin and Vendor users in the linqUp application.

**Note**: All admin users have baseline permissions for login, authentication, password management, 2FA, profile updates, and notifications. All vendor users have baseline permissions for login, authentication, password management, 2FA, profile settings, and notifications. These common permissions are not listed for each role.

---

## Admin Roles

### Super Admin
**Description**: Full access to all admin features and platform management capabilities.

**Permissions**:
- Get revenue dashboard
- View transactions
- View activities
- View orders
- View linqups
- Get escrow
- Global settings earning percentage
- View profiles
- Activities to track
- Add business
- View vendors
- Vendor overview
- Menu
- Orders
- Linqups list calendar
- Wallet
- Stores
- Reviews
- Activities
- Add vendor
- Referrals
- View reports
- Add admin
- Resend invite
- Delete admin
- Deactivate admin
- View subscriptions
- Update pricing
- View analytics
- Audit log

---

### Manager
**Description**: Can manage vendors, orders, and view analytics but cannot modify global settings or pricing.

**Permissions**:
- Get revenue dashboard
- View transactions
- View activities
- View orders
- View linqups
- Get escrow
- View profiles
- Activities to track
- Add business
- View vendors
- Vendor overview
- Menu
- Orders
- Linqups list calendar
- Wallet
- Stores
- Reviews
- Activities
- Add vendor
- Referrals
- View reports
- Add admin
- Resend invite
- Deactivate admin
- View subscriptions
- View analytics
- Audit log

---

### Accountant
**Description**: Focus on financial operations, transactions, and revenue reporting.

**Permissions**:
- Get revenue dashboard
- View transactions
- View activities
- View orders
- Get escrow
- View profiles
- View vendors
- Vendor overview
- Wallet
- Activities
- View reports
- View subscriptions
- View analytics
- Audit log

---

### Member
**Description**: Read-only access to view reports, analytics, and basic information.

**Permissions**:
- View transactions
- View activities
- View orders
- View linqups
- View profiles
- View vendors
- Vendor overview
- Menu
- Orders
- Linqups list calendar
- Wallet
- Stores
- Reviews
- Activities
- View reports
- View subscriptions
- View analytics
- Audit log

---

## Vendor Roles

### Owner
**Description**: Full access to all vendor features and business management capabilities.

**Permissions**:
- Add payout account
- View dashboard
- View orders
- Overview
- Enter booking code
- Customer details
- Menu
- View menus
- Add menu item
- Add menu options
- Wallet
- Deposit
- Escrow
- Withdrawals
- Refunds
- Setup wallet
- View reviews
- Business information
- Business hours
- Delete payout account
- Update payout account
- Add vendor user
- Deactivate vendor user
- Delete vendor user

---

### Manager
**Description**: Can manage orders, menu, and daily operations but cannot manage payout accounts or vendor users.

**Permissions**:
- View dashboard
- View orders
- Overview
- Enter booking code
- Customer details
- Menu
- View menus
- Add menu item
- Add menu options
- Wallet
- Deposit
- Escrow
- Withdrawals
- Refunds
- Setup wallet
- View reviews
- Business information
- Business hours

---

### Accountant
**Description**: Focus on financial operations, wallet management, and transaction reporting.

**Permissions**:
- View dashboard
- View orders
- Overview
- Wallet
- Deposit
- Escrow
- Withdrawals
- Refunds
- Setup wallet
- View reviews

---

### Member
**Description**: Read-only access to view orders, menu, and basic business information.

**Permissions**:
- View dashboard
- View orders
- Overview
- Menu
- View menus
- Wallet
- View reviews
