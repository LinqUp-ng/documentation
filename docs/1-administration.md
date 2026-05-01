# Phase 1: Admin & Vendor Management
*Setting up the ecosystem and internal team controls.*

### 1. Admin Operations
- [ ] **Dashboard:** `GET /admin/activities`, `GET /admin/analytics`, `GET /admin/revenue`.
- [ ] **KYC & Moderation:** `GET /admin/kyc`, `POST /admin/kyc/:id/review`, `GET /admin/reports`.
- [ ] **Team:** `GET /admin/team`, `POST /admin/team` (Add with system-generated password).

### 2. Vendor Onboarding & Teams
- [ ] **Onboarding:** `POST /admin/vendors` (Invite business), `GET /admin/vendors`.
- [ ] **Vendor Team:** `GET /vendor/team`, `POST /vendor/team` (Invite staff with assigned roles).
- [ ] **Permissions:** `GET /vendor/team/permissions` - Fetch role-based access control (RBAC).
