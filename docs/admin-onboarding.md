# Administrative Management

Administrative accounts are controlled via specialized Edge Functions to maintain strict relational integrity and enforce high-level security permissions.

---

## ➕ Adding an Administrator
New administrators are onboarded through the `add-admin` function. This function creates the necessary Auth and Public records in a single transactional flow.

### Security Rules:
- **Authorization**: Only users with the `super_admin` role can invoke this function.
- **Workflow**:
    1. Generates a secure random password.
    2. Creates the Auth user with `admin_role` metadata.
    3. syncs public profile to the `admins` table.
    4. Sends a welcome email with credentials via Resend.

---

## 🚫 Deactivating an Administrator
Administrators can be deactivated using the `deactivate-admin` function. Deactivation acts as a "soft lock" that prevents the user from logging in or performing any API actions.

### 🔒 Critical Security Guards:
To prevent system lockout and unauthorized escalation, the following guards are enforced at the Edge Function level:
1. **Self-Protection**: A user **cannot deactivate themselves**.
2. **Super Admin Guard**: Users with the `super_admin` role **cannot be deactivated** via this function. This ensures that the primary system controllers remain accessible.
3. **Role Check**: Only an active `super_admin` can trigger the deactivation logic.

---

## 🛡️ Role Hierarchy

| Role | Permissions |
| :--- | :--- |
| **Super Admin** | Full access. Can create/deactivate all other admin accounts. |
| **Manager** | Operational access. Can manage vendors and products. |
| **Accountant** | Financial access. Read-only ledger and transaction history. |
| **Member** | Support access. Can respond to user queries and moderate content. |
