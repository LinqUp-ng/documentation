# Onboarding

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

## 📋 Roles API

The roles API provides functions to retrieve role definitions and user counts for role-based access control.

### File: `api/handleRoles.ts`

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
    profile_photo?: any;
    profile_photo_hash?: string;
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

export interface EditAdminParams {
    id: string;
    first_name?: string;
    last_name?: string;
    phone_number?: string;
    profile_photo?: any;
    profile_photo_hash?: string;
    role?: "super_admin" | "manager" | "accountant" | "member";
}

export interface DeactivateAdminParams {
    id: string;
    is_deactivated: boolean;
}

export interface ResendAdminInviteParams {
    email: string;
    is_development?: boolean;
}

/**
 * Add a new administrator via edge function
 */
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

/**
 * Get onboarding admin details via edge function
 * Public function - no auth required
 */
const getOnboardingAdmin = async (token: string): Promise<Response<OnboardingAdminData>> => {
    try {
        const { data, error } = await supabase.functions.invoke("get-onboarding-admin", {
            body: { token },
            headers: {
                Authorization: `Bearer ${supabaseAnonKey}`,
                "x-app-client-secret": appClientSecret,
            },
        });

        console.log("Supabase response:", { data, error });

        if (error) throw error;

        // Check if response contains an error message
        if (data && typeof data === 'object' && 'error' in data) {
            throw new Error(data.error);
        }

        return { data, isSuccessful: true, message: "Fetched onboarding data" };
    } catch (error) {
        console.log("Caught error:", error);
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

/**
 * Complete admin signup by setting password and activating account
 */
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

/**
 * Edit an administrator using Supabase SDK
 */
const editAdmin = async (params: EditAdminParams): Promise<Response<any>> => {
    try {
        const { id, ...updateData } = params;

        const { data, error } = await supabase
            .from("admins")
            .update(updateData)
            .eq("id", id)
            .select("*")
            .single();

        if (error) throw error;

        return { data, isSuccessful: true, message: "Admin updated successfully" };
    } catch (error) {
        throw error;
    }
};

/**
 * Deactivate or activate an administrator by updating the users table
 */
const deactivateAdmin = async (params: DeactivateAdminParams): Promise<Response<any>> => {
    try {
        const { data, error } = await supabase
            .from("users")
            .update({ is_deactivated: params.is_deactivated })
            .eq("id", params.id)
            .select("*")
            .single();

        if (error) throw error;

        return { data, isSuccessful: true, message: params.is_deactivated ? "Admin deactivated successfully" : "Admin activated successfully" };
    } catch (error) {
        throw error;
    }
};

/**
 * Resend admin invitation email via edge function
 */
const resendAdminInvite = async (params: ResendAdminInviteParams): Promise<Response<any>> => {
    try {
        const { data, error } = await supabase.functions.invoke("resend-admin-invite", {
            body: params,
        });

        if (error) throw error;

        return { data, isSuccessful: true, message: "Admin invitation resent successfully" };
    } catch (error) {
        return {
            data: null,
            isSuccessful: false,
            message: error instanceof Error ? error.message : "Failed to resend admin invitation",
        };
    }
};

export const handleAdmin = {
    addAdmin,
    getOnboardingAdmin,
    completeAdminSignup,
    editAdmin,
    deactivateAdmin,
    resendAdminInvite,
};

```

### 1. getAllRoles

Retrieves all roles from the roles table, ordered by user_type and role_type.

```typescript
const getAllRoles = async (): Promise<Response<Role[]>> => {
    try {
        const { data, error } = await supabase
            .from("roles")
            .select("*")
            .order("user_type", { ascending: true })
            .order("role_type", { ascending: true });

        if (error) throw error;

        return { data, isSuccessful: true, message: "Roles fetched successfully" };
    } catch (error) {
        throw error;
    }
};
```

**Usage:**
```typescript
const result = await handleRoles.getAllRoles();
```

### 2. getRoleByUserType

Retrieves roles filtered by user type (admin, vendor, or user).

```typescript
const getRoleByUserType = async (userType: "admin" | "vendor" | "user"): Promise<Response<Role[]>> => {
    try {
        const { data, error } = await supabase
            .from("roles")
            .select("*")
            .eq("user_type", userType)
            .order("role_type", { ascending: true });

        if (error) throw error;

        return { data, isSuccessful: true, message: "Roles fetched successfully" };
    } catch (error) {
        throw error;
    }
};
```

**Usage:**
```typescript
const result = await handleRoles.getRoleByUserType("admin");
```

### 3. getAdminRolesBreakdown

Retrieves admin roles with user count breakdown. This function calls the database function `get_admin_roles_breakdown()` which returns all admin roles with the count of users for each role. Only admins can call this function.

```typescript
const getAdminRolesBreakdown = async (): Promise<Response<CompositeTypes<'admin_roles_breakdown'>[]>> => {
    try {
        const { data, error } = await supabase.rpc("get_admin_roles_breakdown");

        if (error) throw error;

        return { data, isSuccessful: true, message: "Admin roles breakdown fetched successfully" };
    } catch (error) {
        throw error;
    }
};
```

**Usage:**
```typescript
const result = await handleRoles.getAdminRolesBreakdown();
```

**Security Note:** This function can only be called by authenticated admin users. The database function enforces this restriction.

### Export

```typescript
export const handleRoles = {
    getAllRoles,
    getRoleByUserType,
    getAdminRolesBreakdown,
};
```

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

export interface EditAdminParams {
    id: string;
    first_name?: string;
    last_name?: string;
    phone_number?: string;
    profile_photo?: any;
    profile_photo_hash?: string;
    role?: "super_admin" | "manager" | "accountant" | "member";
}

export interface DeactivateAdminParams {
    id: string;
    is_deactivated: boolean;
}

export interface ResendAdminInviteParams {
    email: string;
    is_development?: boolean;
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

**Note:** After successfully completing signup, the user should call the standard login endpoint to establish their session. See `api/handleAuth.ts` for the login implementation.

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

### 4. editAdmin

Edits an administrator's profile information using Supabase SDK. This function updates the `admins` table directly.

```typescript
const editAdmin = async (params: EditAdminParams): Promise<Response<any>> => {
    try {
        const { id, ...updateData } = params;

        const { data, error } = await supabase
            .from("admins")
            .update(updateData)
            .eq("id", id)
            .select("*")
            .single();

        if (error) throw error;

        return { data, isSuccessful: true, message: "Admin updated successfully" };
    } catch (error) {
        throw error;
    }
};
```

**Usage:**
```typescript
const result = await handleAdmin.editAdmin({
    id: "admin-uuid-here",
    first_name: "Jane",
    last_name: "Smith",
    role: "manager"
});
```

**Notes:**
- Only the fields provided in the request will be updated
- The `id` field is required to identify which admin to update
- Role changes should be done carefully as they affect permissions

### 5. deactivateAdmin

Deactivates or activates an administrator by updating the `users` table. Deactivation acts as a "soft lock" that prevents the user from logging in or performing any API actions.

```typescript
const deactivateAdmin = async (params: DeactivateAdminParams): Promise<Response<any>> => {
    try {
        const { data, error } = await supabase
            .from("users")
            .update({ is_deactivated: params.is_deactivated })
            .eq("id", params.id)
            .select("*")
            .single();

        if (error) throw error;

        return { data, isSuccessful: true, message: params.is_deactivated ? "Admin deactivated successfully" : "Admin activated successfully" };
    } catch (error) {
        throw error;
    }
};
```

**Usage:**
```typescript
// Deactivate an admin
const result = await handleAdmin.deactivateAdmin({
    id: "admin-uuid-here",
    is_deactivated: true
});

// Reactivate an admin
const result = await handleAdmin.deactivateAdmin({
    id: "admin-uuid-here",
    is_deactivated: false
});
```

**Security Guards:**
- A user cannot deactivate themselves
- Users with the `super_admin` role cannot be deactivated via this function
- Only an active `super_admin` can trigger the deactivation logic

### 6. resendAdminInvite

Resends an admin invitation email via the `resend-admin-invite` edge function. This is useful when an admin needs to receive their onboarding invitation again.

```typescript
const resendAdminInvite = async (params: ResendAdminInviteParams): Promise<Response<any>> => {
    try {
        const { data, error } = await supabase.functions.invoke("resend-admin-invite", {
            body: params,
        });

        if (error) throw error;

        return { data, isSuccessful: true, message: "Admin invitation resent successfully" };
    } catch (error) {
        return {
            data: null,
            isSuccessful: false,
            message: error instanceof Error ? error.message : "Failed to resend admin invitation",
        };
    }
};
```

**Usage:**
```typescript
const result = await handleAdmin.resendAdminInvite({
    email: "admin@example.com",
    is_development: false
});

if (result.isSuccessful) {
    console.log("Admin invitation resent successfully");
} else {
    console.error("Failed to resend invitation:", result.message);
}
```

**Notes:**
- The function requires the app client secret in headers for security
- Use `is_development: true` when testing in development environment
- The admin will receive a new onboarding link via email

### Export

```typescript
export const handleAdmin = {
    addAdmin,
    getOnboardingAdmin,
    completeAdminSignup,
    editAdmin,
    deactivateAdmin,
    resendAdminInvite,
};
```
