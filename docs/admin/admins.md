# Admin Management

This document covers administrator management operations for the platform.

## API Implementation

### File: `api/handleAdmin.ts`

```typescript
import { supabase, Tables } from "@/lib/supabase";
import { Response } from "@/types/api";
import { CompositeTypes, TablesUpdate } from "@/types/database.types";

const supabaseAnonKey = process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY || "";
const appClientSecret = process.env.EXPO_PUBLIC_APP_CLIENT_SECRET || "";
```

---

## Types

### Admin Management Types

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
```

### Admin Operations Types

```typescript
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

### Admin Query Types

```typescript
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

## Functions

### 1. addAdmin

Creates a new administrator via edge function. This operation requires super_admin authentication.

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

**Endpoint:** `POST /functions/v1/add-admin`

**Request Body (AddAdminParams):**
```typescript
{
    first_name: string;
    last_name: string;
    email: string;
    role: "super_admin" | "manager" | "accountant" | "member";
    phone_number?: string;
    profile_photo?: { url: string; blur_hash?: string } | null;
    profile_photo_hash?: string;
}
```

**Usage Example:**
```typescript
const result = await handleAdmin.addAdmin({
    first_name: "Jane",
    last_name: "Doe",
    email: "jane.doe@linqup.com",
    role: "manager",
    phone_number: "+2348012345678"
});

if (result.isSuccessful) {
    console.log("Admin added successfully");
    // The new admin will receive an invitation email
}
```

**Notes:**
- The new admin will receive an invitation email with a link to set their password
- Profile photo should be uploaded first using the upload API
- Only super_admins can add new administrators

---

### 2. getOnboardingAdmin

Retrieves the admin's onboarding information using the token provided. This is a public endpoint that does not require authentication but requires the app client secret.

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
const result = await handleAdmin.getOnboardingAdmin(token);

if (result.isSuccessful) {
    console.log("Email:", result.data.email);
    console.log("Role:", result.data.role);
    console.log("Name:", result.data.first_name, result.data.last_name);
}
```

**Notes:**
- This function is used during the admin onboarding flow
- The token is provided via the invitation email link
- See [Admin Onboarding](/docs/admin/onboarding) for the complete onboarding flow

---

### 3. completeAdminSignup

Completes the admin signup by setting a password and activating the account.

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

**Endpoint:** `POST /functions/v1/complete-admin-signup`

**Request Body (CompleteAdminSignupParams):**
```typescript
{
    token: string;    // The onboarding token
    password: string; // The new password (must meet security requirements)
}
```

**Usage Example:**
```typescript
const result = await handleAdmin.completeAdminSignup({
    token: onboardingToken,
    password: "securePassword123"
});

if (result.isSuccessful) {
    console.log("Account activated for user:", result.data.userId);
    // Now call login to establish session
}
```

**Notes:**
- The password must meet the platform's security requirements
- After successful signup, the admin should log in to establish a session
- See [Admin Onboarding](/docs/admin/onboarding) for the complete onboarding flow

---

### 4. editAdmin

Edits an administrator's information using Supabase SDK. This operation requires appropriate authentication.

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

**Usage Example:**
```typescript
// Update admin profile
const result = await handleAdmin.editAdmin({
    id: 'admin-uuid-here',
    first_name: 'Jane',
    last_name: 'Smith',
    phone_number: '+2348012345678'
});

if (result.isSuccessful) {
    console.log("Admin updated successfully");
    console.log("Updated admin:", result.data);
}

// Update admin role (super_admin only)
const roleUpdate = await handleAdmin.editAdmin({
    id: 'admin-uuid-here',
    role: 'manager'
});
```

**Notes:**
- Admins can typically update their own profile information
- Role changes usually require super_admin privileges
- Profile photo should be uploaded first using the upload API

---

### 5. deactivateAdmin

Deactivates or activates an administrator by updating the users table. This operation requires super_admin authentication.

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

**Usage Example:**
```typescript
// Deactivate an admin
const result = await handleAdmin.deactivateAdmin({
    id: 'admin-uuid-here',
    is_deactivated: true
});

if (result.isSuccessful) {
    console.log("Admin deactivated successfully");
}

// Reactivate an admin
const reactivateResult = await handleAdmin.deactivateAdmin({
    id: 'admin-uuid-here',
    is_deactivated: false
});

if (reactivateResult.isSuccessful) {
    console.log("Admin activated successfully");
}
```

**Notes:**
- Deactivated admins cannot log in or access the system
- This action is reversible - set `is_deactivated: false` to reactivate
- Only super_admins can deactivate other administrators

---

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

**Usage Example:**
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
- Use `is_development: true` when testing in development environment
- The admin will receive a new onboarding link via email
- Only super_admins can resend admin invitations

---

### 7. searchAdmins

Searches admins with fuzzy search and pagination via database function. This operation requires admin authentication.

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

**Usage Example:**
```typescript
// Basic search
const result = await handleAdmin.searchAdmins({
    searchQuery: 'john',
    limit: 10
});

if (result.isSuccessful) {
    const admins = result.data;
    admins.forEach(admin => {
        console.log(admin.first_name, admin.last_name, admin.role);
        console.log('Email:', admin.email);
        console.log('Last Sign In:', admin.last_sign_in_at);
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
    searchQuery?: string;        // Optional: Fuzzy search on name/email
    limit?: number;              // Optional: Results per page (default: 10)
    cursorFirstName?: string;    // Optional: Cursor for pagination
    cursorId?: string;           // Optional: Cursor for pagination
    startDate?: string;          // Optional: Filter by created_at start (ISO 8601)
    endDate?: string;            // Optional: Filter by created_at end (ISO 8601)
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

**Usage Example:**
```typescript
// Get overall summary
const result = await handleAdmin.getAdminSummary({});

if (result.isSuccessful) {
    const summary = result.data;
    console.log('Total admins:', summary.all_admin_count);
    console.log('Active admins:', summary.active_admin_count);
    console.log('Pending activation:', summary.pending_activation_admin_count);
}

// Get summary for a specific period
const monthlySummary = await handleAdmin.getAdminSummary({
    startDate: '2024-01-01T00:00:00Z',
    endDate: '2024-01-31T23:59:59Z'
});

// Get summary with search filter
const filteredSummary = await handleAdmin.getAdminSummary({
    searchQuery: 'manager'
});
```

**Response Type:**
```typescript
{
    data: {
        all_admin_count: number;
        active_admin_count: number;
        pending_activation_admin_count: number;
    };
    isSuccessful: boolean;
    message: string;
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

        if (error) throw error;

        return { data, isSuccessful: true, message: "Admins exported successfully" };
    } catch (error) {
        throw error;
    }
};
```

**Usage Example:**
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
    startDate?: string;          // Optional: Filter by created_at start (ISO 8601)
    endDate?: string;            // Optional: Filter by created_at end (ISO 8601)
    isPendingActivation?: boolean; // Optional: Filter by pending activation status
    searchQuery?: string;        // Optional: Fuzzy search filter
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

**Usage Example:**
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

## Export

```typescript
export const handleAdmin = {
    addAdmin,
    getOnboardingAdmin,
    completeAdminSignup,
    editAdmin,
    deactivateAdmin,
    resendAdminInvite,
    searchAdmins,
    getAdminSummary,
    exportAdmins,
    getAdminById,
};
```

---

## Security Notes

- **Super Admin Functions**: `addAdmin`, `deactivateAdmin`, and `resendAdminInvite` require `super_admin` role
- **Admin Functions**: `searchAdmins`, `getAdminSummary`, and `exportAdmins` require admin authentication
- **Onboarding Functions**: `getOnboardingAdmin` and `completeAdminSignup` are public but require the app client secret header
- **Self-Service**: `editAdmin` can typically be used by admins to update their own profiles
- All functions use `security definer` database functions where applicable and perform role checks internally

---

## Admin Roles

The platform supports the following admin roles:

| Role | Description | Permissions |
|------|-------------|-------------|
| `super_admin` | Full system access | All admin operations including managing other admins |
| `manager` | Operational management | Manage vendors, view analytics, handle support |
| `accountant` | Financial operations | View transactions, manage withdrawals, financial reports |
| `member` | Basic access | View-only access to assigned areas |