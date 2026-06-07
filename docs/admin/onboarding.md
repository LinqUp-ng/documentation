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
    3. Syncs public profile to the `admins` table.
    4. Sends a welcome email with credentials via Resend.

---

## 🚫 Deactivating an Administrator
Administrators can be deactivated using the `deactivateAdmin` function. Deactivation acts as a "soft lock" that prevents the user from logging in or performing any API actions.

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

## 📋 Admin API Reference

### File: `api/handleAdmin.ts`

```typescript
import { supabase } from "@/lib/supabase";
import { Response } from "@/types/api";
import { CompositeTypes, TablesUpdate } from "@/types/database.types";

const supabaseAnonKey = process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY || "";
const appClientSecret = process.env.EXPO_PUBLIC_APP_CLIENT_SECRET || "";
```

### Types

```typescript
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

export interface SearchAdminsParams {
    searchQuery?: string;
    limit?: number;
    cursorFirstName?: string;
    cursorId?: string;
    startDate?: string;
    endDate?: string;
    isPendingActivation?: boolean;
}

export interface AdminItemResponse {
    id: string;
    email: string;
    first_name: string | null;
    last_name: string | null;
    role: string | null;
    last_sign_in_at: string | null;
    is_deactivated: boolean;
    is_pending_activation: boolean | null;
}

export interface AdminSummaryResponse {
    all_admin_count: number;
    active_admin_count: number;
    pending_activation_admin_count: number;
}

export interface ExportAdminsParams {
    startDate?: string;
    endDate?: string;
    isPendingActivation?: boolean;
    searchQuery?: string;
}
```

---

### 1. addAdmin

Adds a new administrator via the `add-admin` edge function.

```typescript
const addAdmin = async (params: AddAdminParams): Promise<Response<any>> => {
    try {
        const { data, error } = await supabase.functions.invoke("add-admin", {
            body: params,
        });

        if (error) {
            const errorBody = await error.context?.json();
            console.log("Detailed Function Error:", errorBody);
            throw error;
        }


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

---

### 2. getOnboardingAdmin

Retrieves onboarding admin details via the `get-onboarding-admin` edge function. This is a public function that requires no authentication but requires the app client secret.

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

        if (error) {
            const errorBody = await error.context?.json();
            console.log("Detailed Function Error:", errorBody);
            throw error;
        }


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

if (result.isSuccessful) {
    console.log("Email:", result.data.email);
    console.log("Role:", result.data.role);
    console.log("Name:", result.data.first_name, result.data.last_name);
}
```

---

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

        if (error) {
            const errorBody = await error.context?.json();
            console.log("Detailed Function Error:", errorBody);
            throw error;
        }


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

if (result.isSuccessful && result.data.success) {
    console.log("User ID:", result.data.userId);
    // Now call login to establish session
}
```

---

### 4. editAdmin

Edits an administrator's profile information using Supabase SDK. This function updates the `admins` table directly.

```typescript
const editAdmin = async (params: TablesUpdate<'admins'>): Promise<Response<any>> => {
    try {
        const { id, ...updateData } = params;

        if (!id) {
            throw new Error("Admin ID is required");
        }

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

---

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

---

### 6. resendAdminInvite

Resends an admin invitation email via the `resend-admin-invite` edge function. This is useful when an admin needs to receive their onboarding invitation again.

```typescript
const resendAdminInvite = async (params: ResendAdminInviteParams): Promise<Response<any>> => {
    try {
        const { data, error } = await supabase.functions.invoke("resend-admin-invite", {
            body: params,
        });

        if (error) {
            const errorBody = await error.context?.json();
            console.log("Detailed Function Error:", errorBody);
            throw error;
        }


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

---

### 7. searchAdmins

Searches admins with fuzzy text search and pagination via database function. This operation requires admin authentication.

```typescript
const searchAdmins = async (params: SearchAdminsParams): Promise<Response<CompositeTypes<'admin_item_response'>[]>> => {
    try {
        const { data, error } = await supabase.rpc('search_admins', {
            p_limit: params.limit || 10,
            ...(params.startDate && {p_start_date: params.startDate}),
            ...(params.endDate && {p_end_date: params.endDate}),
            ...(params.searchQuery && {p_search_query: params.searchQuery}),
            ...(params.cursorFirstName && {p_cursor_first_name: params.cursorFirstName}),
            ...(params.cursorId && {p_cursor_id: params.cursorId}),
            ...(params.isPendingActivation !== undefined && {p_is_pending_activation: params.isPendingActivation}),
            p_include_all: false
        });

        if (error) throw error;

        return { data: data || [], isSuccessful: true, message: "Admins retrieved successfully" };
    } catch (error) {
        return {
            data: [],
            isSuccessful: false,
            message: error instanceof Error ? error.message : "Failed to search admins",
        };
    }
};
```

**Usage:**
```typescript
// Basic search
const result = await handleAdmin.searchAdmins({
    searchQuery: 'john',
    limit: 10
});

if (result.isSuccessful) {
    const admins = result.data;
    admins.forEach(admin => {
        console.log(admin.email, admin.first_name, admin.last_name, admin.role);
    });
}

// Filter by pending activation status
const pendingAdmins = await handleAdmin.searchAdmins({
    isPendingActivation: true,
    limit: 10
});

// Paginated search with cursor
const lastAdmin = admins[admins.length - 1];
const nextPage = await handleAdmin.searchAdmins({
    searchQuery: 'john',
    limit: 10,
    cursorFirstName: lastAdmin.first_name,
    cursorId: lastAdmin.id
});

// Filter by date range
const recentAdmins = await handleAdmin.searchAdmins({
    startDate: '2024-01-01T00:00:00Z',
    endDate: '2024-01-31T23:59:59Z',
    limit: 10
});
```

**Request Parameters (SearchAdminsParams):**
```typescript
{
    searchQuery?: string;          // Optional: Fuzzy search on name/email
    limit?: number;                // Optional: Results per page (default: 10)
    cursorFirstName?: string;      // Optional: Cursor for pagination
    cursorId?: string;             // Optional: Cursor for pagination
    startDate?: string;            // Optional: Filter by created_at start (ISO 8601)
    endDate?: string;              // Optional: Filter by created_at end (ISO 8601)
    isPendingActivation?: boolean; // Optional: Filter by pending activation status
}
```

**Response Type:**
```typescript
{
    data: CompositeTypes<'admin_item_response'>[];
    isSuccessful: boolean;
    message: string;
}
```

**Admin Item Response:**
```typescript
{
    id: string;
    email: string;
    first_name: string | null;
    last_name: string | null;
    role: string | null;
    last_sign_in_at: string | null;
    is_deactivated: boolean;
    is_pending_activation: boolean | null;
}
```

---

### 8. getAdminSummary

Retrieves admin summary statistics via database function. This operation requires admin authentication.

```typescript
const getAdminSummary = async (params: Omit<SearchAdminsParams, 'limit' | 'cursorFirstName' | 'cursorId'>): Promise<Response<CompositeTypes<'admin_summary_response'>>> => {
    try {
        const { data, error } = await supabase.rpc('get_all_admin_summary', {
            ...(params.startDate && {p_start_date: params.startDate}),
            ...(params.endDate && {p_end_date: params.endDate}),
            ...(params.searchQuery && {p_search_query: params.searchQuery}),
        });

        if (error) throw error;

        return { data, isSuccessful: true, message: "Admin summary retrieved successfully" };
    } catch (error) {
        return {
            data: {} as AdminSummaryResponse,
            isSuccessful: false,
            message: error instanceof Error ? error.message : "Failed to get admin summary",
        };
    }
};
```

**Usage:**
```typescript
const result = await handleAdmin.getAdminSummary({
    startDate: '2024-01-01T00:00:00Z',
    endDate: '2024-01-31T23:59:59Z',
    searchQuery: 'manager'
});

if (result.isSuccessful) {
    const summary = result.data;
    console.log('Total admins:', summary.all_admin_count);
    console.log('Active admins:', summary.active_admin_count);
    console.log('Pending activation:', summary.pending_activation_admin_count);
}
```

**Response Type:**
```typescript
{
    data: AdminSummaryResponse;
    isSuccessful: boolean;
    message: string;
}
```

**Admin Summary Response:**
```typescript
{
    all_admin_count: number;
    active_admin_count: number;
    pending_activation_admin_count: number;
}
```

---

### 9. exportAdmins

Exports admins to Excel via edge function. This operation requires admin authentication.

```typescript
const exportAdmins = async (params: ExportAdminsParams): Promise<Response<{ path: string; url: string }>> => {
    try {
        const { data, error } = await supabase.functions.invoke("export-admins", {
            body: params,
        });

        if (error) {
            const errorBody = await error.context?.json();
            console.log("Detailed Function Error:", errorBody);
            throw error;
        }


        return { data, isSuccessful: true, message: "Admins exported successfully" };
    } catch (error) {
        throw error;
    }
};
```

**Usage:**
```typescript
const result = await handleAdmin.exportAdmins({
    startDate: '2024-01-01T00:00:00Z',
    endDate: '2024-01-31T23:59:59Z',
    isPendingActivation: false,
    searchQuery: 'manager'
});

if (result.isSuccessful) {
    console.log('Export file path:', result.data.path);
    console.log('Download URL:', result.data.url);
    // Open or download the file from the URL
}
```

**Request Parameters (ExportAdminsParams):**
```typescript
{
    startDate?: string;      // Optional: Filter by created_at start (ISO 8601)
    endDate?: string;        // Optional: Filter by created_at end (ISO 8601)
    isPendingActivation?: boolean; // Optional: Filter by pending activation status
    searchQuery?: string;    // Optional: Fuzzy search filter
}
```

**Response:**
```typescript
{
    path: string;  // Path to the exported file in storage
    url: string;   // Pre-signed URL to download the file
}
```

---

### 10. getAdminById

Retrieves an admin by their unique identifier with all columns.

```typescript
const getAdminById = async (id: string): Promise<Response<Tables<'admins'> | null>> => {
    try {
        const { data, error } = await supabase
            .from("admins")
            .select("*")
            .eq("id", id)
            .single();

        if (error) throw error;

        return { data, isSuccessful: true, message: "Admin retrieved successfully" };
    } catch (error) {
        return {
            data: null,
            isSuccessful: false,
            message: error instanceof Error ? error.message : "Failed to fetch admin",
        };
    }
};
```

**Usage:**
```typescript
const result = await handleAdmin.getAdminById('admin-uuid-here');

if (result.isSuccessful && result.data) {
    console.log('Admin:', result.data.first_name, result.data.last_name);
    console.log('Email:', result.data.email);
    console.log('Role:', result.data.role);
    console.log('Phone:', result.data.phone_number);
}
```

**Response Type:**
```typescript
{
    data: Tables<'admins'> | null;
    isSuccessful: boolean;
    message: string;
}
```

---

## 📋 Roles API Reference

### File: `api/handleRoles.ts`

```typescript
import { supabase } from "@/lib/supabase";
import { Response } from "@/types/api";
import { CompositeTypes } from "@/types/database.types";
```

### Types

```typescript
export interface Role {
    id: string;
    role_type: "super_admin" | "owner" | "manager" | "accountant" | "member";
    user_type: "user" | "vendor" | "admin";
    readable_role_name: string;
    description: string;
    permissions: string[];
    created_at: string;
    updated_at: string;
}
```

---

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

if (result.isSuccessful) {
    result.data.forEach(role => {
        console.log(`${role.role_type} (${role.user_type}): ${role.description}`);
        console.log('Permissions:', role.permissions.join(', '));
    });
}
```

---

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
// Get all admin roles
const adminRoles = await handleRoles.getRoleByUserType("admin");

// Get all vendor roles
const vendorRoles = await handleRoles.getRoleByUserType("vendor");

// Get all user roles
const userRoles = await handleRoles.getRoleByUserType("user");
```

---

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

if (result.isSuccessful) {
    result.data.forEach(role => {
        console.log(`${role.role_type}: ${role.user_count} users`);
    });
}
```

**Security Note:** This function can only be called by authenticated admin users. The database function enforces this restriction.

---

### 4. getStoreRolesBreakdown

Retrieves store/vendor roles with user count breakdown for a specific store. This function calls the database function `get_store_roles_breakdown(store_id)` which returns all vendor roles (owner, manager, accountant, member) with the count of users for each role in the specified store. Only admins or vendors with access to the store can call this function.

```typescript
const getStoreRolesBreakdown = async (storeId: string): Promise<Response<CompositeTypes<'store_roles_breakdown'>[]>> => {
    try {
        const { data, error } = await supabase.rpc("get_store_roles_breakdown", { p_store_id: storeId });

        if (error) throw error;

        return { data, isSuccessful: true, message: "Store roles breakdown fetched successfully" };
    } catch (error) {
        throw error;
    }
};
```

**Usage:**
```typescript
const storeId = "store-uuid-here";
const result = await handleRoles.getStoreRolesBreakdown(storeId);

if (result.isSuccessful) {
    result.data.forEach(role => {
        console.log(`${role.role_type}: ${role.user_count} users`);
    });
}
```

**Security Note:** This function can only be called by authenticated admin users or vendors who have access to the specified store. The database function enforces this restriction.

---

### Export

```typescript
export const handleRoles = {
    getAllRoles,
    getRoleByUserType,
    getAdminRolesBreakdown,
    getStoreRolesBreakdown,
};
```

---

## 🛡️ Security: Shared Secret Verification

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