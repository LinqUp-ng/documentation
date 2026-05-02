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

---

## � Security: Shared Secret Verification

The public onboarding functions (`get-onboarding-admin`, `complete-admin-signup`, `get-onboarding-vendor`, `complete-vendor-signup`) require a shared secret to ensure requests originate from the legitimate app client.

**Server-side (Supabase Edge Functions):**
- Environment variable: `APP_CLIENT_SECRET`
- Header to check: `x-app-client-secret`
- Returns 401 Unauthorized if missing or mismatched

**Client-side (App):**
- Environment variable: `EXPO_PUBLIC_APP_CLIENT_SECRET`
- Sent in headers: `x-app-client-secret`

**Setup:**
1. Set `APP_CLIENT_SECRET` as a secret in your Supabase project
2. Add `EXPO_PUBLIC_APP_CLIENT_SECRET` to your `.env` file with the same value

---

## � API Implementation

### File: `api/handleAdmin.ts`

```typescript
import { supabase } from "@/lib/supabase";
import { Response } from "@/types/api";

const supabaseAnonKey = process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY || "";
const appClientSecret = process.env.EXPO_PUBLIC_APP_CLIENT_SECRET || "";

export interface AddAdminParams {
    first_name: string;
    last_name: string;
    email: string;
    role: "super_admin" | "manager" | "accountant" | "member";
    phone_number?: string;
}

export interface OnboardingAdminData {
    email: string;
    role: string;
    first_name: string | null;
    last_name: string | null;
    created_at: string;
}

export interface CompleteAdminSignupParams {
    token: string;
    password: string;
}

export interface CompleteAdminSignupResponse {
    success: boolean;
    message: string;
    userId: string;
    session?: any;
    user?: any;
}
```

### 1. addAdmin

Adds a new administrator via the `add-admin` edge function.

```typescript
const addAdmin = async (params: AddAdminParams): Promise<Response<any>> => {
    try {
        const { data, error } = await supabase.functions.invoke("add-admin", {
            body: params,
        });

        if (error) throw error;

        return { data, isSuccessful: true, message: "Admin added successfully" };
    } catch (error) {
        return {
            data: null,
            isSuccessful: false,
            message: error instanceof Error ? error.message : "Failed to add administrator",
        };
    }
};
```

**Usage:**
```typescript
const result = await handleAdmin.addAdmin({
    first_name: "John",
    last_name: "Doe",
    email: "john@example.com",
    role: "manager",
    phone_number: "+1234567890"
});
```

### 2. getOnboardingAdmin

Retrieves onboarding admin details via the `get-onboarding-admin` edge function. This is a public function that requires no authentication.

```typescript
const getOnboardingAdmin = async (token: string): Promise<Response<OnboardingAdminData>> => {
    try {
        const { data, error } = await supabase.functions.invoke("get-onboarding-admin", {
            body: { token },
            headers: {
                Authorization: `Bearer ${supabaseAnonKey}`,
                "x-app-client-secret": appClientSecret,
            },
        });

        if (error) throw error;

        // Check if response contains an error message
        if (data && typeof data === 'object' && 'error' in data) {
            throw new Error(data.error);
        }

        return { data, isSuccessful: true, message: "Fetched onboarding data" };
    } catch (error) {
        let errorMessage = "Failed to fetch onboarding data";
        if (error instanceof Error) {
            errorMessage = error.message;
        } else if (typeof error === 'string') {
            errorMessage = error;
        } else if (error && typeof error === 'object' && 'message' in error) {
            errorMessage = String(error.message);
        }
        return {
            data: {} as OnboardingAdminData,
            isSuccessful: false,
            message: errorMessage,
        };
    }
};
```

**Usage:**
```typescript
const result = await handleAdmin.getOnboardingAdmin("abc123token");
```

### 3. completeAdminSignup

Completes admin signup by setting a password and activating the account via the `complete-admin-signup` edge function.

```typescript
const completeAdminSignup = async (
    params: CompleteAdminSignupParams
): Promise<Response<CompleteAdminSignupResponse>> => {
    try {
        const { data, error } = await supabase.functions.invoke("complete-admin-signup", {
            body: params,
            headers: {
                Authorization: `Bearer ${supabaseAnonKey}`,
                "x-app-client-secret": appClientSecret,
            },
        });

        if (error) throw error;

        return { data, isSuccessful: true, message: "Signup completed successfully" };
    } catch (error) {
        return {
            data: {} as CompleteAdminSignupResponse,
            isSuccessful: false,
            message: error instanceof Error ? error.message : "Failed to complete signup",
        };
    }
};
```

**Usage:**
```typescript
const result = await handleAdmin.completeAdminSignup({
    token: "abc123token",
    password: "SecurePassword123!"
});
```

### Export

```typescript
export const handleAdmin = {
    addAdmin,
    getOnboardingAdmin,
    completeAdminSignup,
};
```
