# OTP & Messaging Requirements

This document defines all scenarios in the LinqUp ecosystem that require the dispatch of **One-Time Passwords (OTP)**, **System-Generated Credentials**, or **Time-Sensitive Notifications**.

---

## 🔐 Security & Authentication (OTP)

These actions require a 6-digit code sent via email or SMS to verify the user's identity.

| Action | Purpose | Channel |
|---|---|---|
| **Create Account** | Verify email/phone during registration. | Email/SMS |
| **Enable 2FA** | Linking an account to a secondary verification method. | Email |
| **Reset Password** | Verifying identity after a "Forgot Password" request. | Email |
| **Payout Account Change** | Guarding sensitive financial destination changes. | Email |
| **Refund Wallet** | Affirming a wallet-to-source refund request. | Email |

---

## 📧 Administrative & Onboarding

Automated emails sent during platform management and team expansion.

### 1. New Admin Onboarding
*   **Trigger**: Super Admin creates a new Admin in the dashboard.
*   **Content**: Welcome email containing a **system-generated temporary password** and a link to the Admin portal.
*   **Requirement**: User must be forced to change password on first login.

### 2. New Vendor Onboarding
*   **Trigger**: Admin approves a Vendor KYC or manually adds a Vendor.
*   **Content**: Credential setup link and a guide on setting up their restaurant profile.

### 3. Vendor Team Management
*   **Trigger**: A Vendor Manager adds a new staff member (Team Member).
*   **Content**: Invite link with pre-assigned role permissions.

---

## ⚙️ Role & Transactional Alerts

Notifications triggered by system changes or successful operations.

| Action | Notification Type | Content |
|---|---|---|
| **Role Changes** | Security Alert | "Your account role has been updated to [Role]. If this wasn't you, contact support." |
| **Transaction Completed** | Receipt | "Payment of [Amount] to [Vendor] was successful. Order Code: [Code]." |
| **Wallet Funded** | Transaction Alert | "Your wallet has been credited with [Amount]." |

---

## 🗓️ Scheduled Reminders (Dating)

Automated pings to ensure users don't miss their LinqUps.

### **24-Hour Date Reminder**
*   **Trigger**: 24 hours before the scheduled `match.date_time`.
*   **Recipient**: Both match participants.
*   **Content**: "You have a LinqUp tomorrow at [Time] with [Name] at [Restaurant Name]! Check your itinerary for details."
*   **Backend Logic**: Handled by a scheduled Edge Function or pg_cron worker.
