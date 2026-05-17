# Vendor Management

This document covers vendor-related operations for administrators.

## API Implementation

### File: `api/handleVendors.ts`

```typescript
import { Enums, supabase } from "@/lib/supabase";
import { Response } from "@/types/api";
import { CompositeTypes, Database } from "@/types/database.types";
import { UploadWithBlurhashResponse } from "./handleUpload";

const supabaseAnonKey = process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY || "";
const appClientSecret = process.env.EXPO_PUBLIC_APP_CLIENT_SECRET || "";
```

---

## Types

### Working Hours

```typescript
export type DayOfWeek = Database["public"]["Enums"]["day_of_the_week"];

export interface WorkingHour {
    day_of_the_week: DayOfWeek;
    opening_time: string;
    closing_time: string;
    is_opened?: boolean;
}
```

### Onboarding Types

```typescript
export interface OnboardingVendorData {
    email: string;
    business_name: string;
    business_status: string;
    first_name: string | null;
    last_name: string | null;
    vendor_role: string;
    created_at: string;
}

export interface CompleteVendorSignupParams {
    token: string;
    password: string;
}

export interface CompleteVendorSignupResponse {
    success: boolean;
    message: string;
    userId: string;
    session?: any;
    user?: any;
}
```

### Business Management Types

```typescript
export interface AddBusinessParams {
    business_name: string;
    business_address?: string;
    cac_number?: string;
    tax_id?: string;
    cac_document_url?: string;
    store_name: string;
    store_address?: string;
    store_city?: string;
    store_state?: string;
    store_latitude?: number;
    store_longitude?: number;
    store_logo?: Omit<UploadWithBlurhashResponse, 'isSuccessful' | 'message'> | null;
    store_cover_photo?: any;
    store_cover_photo_blur_hash?: string;
    vendor_first_name: string;
    vendor_last_name: string;
    vendor_phone_number?: string;
    vendor_profile_photo?: any;
    vendor_profile_photo_hash?: string;
    vendor_email: string;
    working_hours?: WorkingHour[];
    is_development?: boolean;
}

export interface AddBusinessResponse {
    success: boolean;
    businessId: string;
    storeId: string;
    vendorUserId: string;
    onboardingToken: string;
}

export interface AddVendorParams {
    vendor_first_name: string;
    vendor_last_name: string;
    vendor_phone_number?: string;
    vendor_profile_photo?: any;
    vendor_profile_photo_hash?: string;
    vendor_email: string;
    vendor_role?: "owner" | "manager" | "member";
    business_id: string;
    store_id: string;
    is_development?: boolean;
}

export interface AddVendorResponse {
    success: boolean;
    businessId: string;
    vendorUserId: string;
    onboardingToken: string;
}

export interface ResendVendorInviteParams {
    email: string;
    is_development?: boolean;
}
```

### Business Query Types

```typescript
export interface GetAllBusinessesParams {
    limit?: number;
    cursorBusinessName?: string;
    cursorId?: string;
    startDate?: string;
    endDate?: string;
    status?: Database["public"]["Enums"]["business_status_enum"];
}

export interface SearchBusinessParams {
    searchQuery?: string;
    limit?: number;
    cursorBusinessName?: string;
    cursorId?: string;
    startDate?: string;
    endDate?: string;
    status?: Database["public"]["Enums"]["business_status_enum"];
}

export interface GetAllBusinessSummaryParams {
    status?: Database["public"]["Enums"]["business_status_enum"];
    startDate?: string;
    endDate?: string;
    searchQuery?: string;
}
```

### Vendor Query Types

```typescript
export interface SearchVendorsParams {
    storeId: string;
    searchQuery?: string;
    limit?: number;
    cursorFirstName?: string;
    cursorId?: string;
    startDate?: string;
    endDate?: string;
    isPendingActivation?: boolean;
}

export interface GetAllVendorSummaryParams {
    storeId: string;
    isPendingActivation?: boolean;
    startDate?: string;
    endDate?: string;
    searchQuery?: string;
}

export interface ExportVendorsParams {
    storeId: string;
    startDate?: string;
    endDate?: string;
    isPendingActivation?: boolean;
    searchQuery?: string;
}

export interface ExportVendorsResponse {
    path: string;
    url: string;
}
```

---

## Functions

### 1. getOnboardingVendor

Retrieves the vendor's onboarding information using the token provided by the administrator. This is a public endpoint that does not require authentication but requires the app client secret.

```typescript
const getOnboardingVendor = async (token: string): Promise<Response<OnboardingVendorData>> => {
    try {
        const { data, error } = await supabase.functions.invoke("get-onboarding-vendor", {
            body: { token },
            headers: {
                Authorization: `Bearer ${supabaseAnonKey}`,
                "x-app-client-secret": appClientSecret,
            },
        });

        if (error) throw error;

        return { data, isSuccessful: true, message: "Fetched onboarding data" };
    } catch (error) {
        return {
            data: {} as OnboardingVendorData,
            isSuccessful: false,
            message: error instanceof Error ? error.message : "Failed to fetch onboarding data",
        };
    }
};
```

**Usage:**
```typescript
const result = await handleVendors.getOnboardingVendor(token);

if (result.isSuccessful) {
    console.log("Business:", result.data.business_name);
    console.log("Email:", result.data.email);
    console.log("Role:", result.data.vendor_role);
    console.log("Name:", result.data.first_name, result.data.last_name);
}
```

See [Vendor Onboarding](/docs/vendor/onboarding) for the complete onboarding flow.

---

### 2. completeVendorSignup

Completes the vendor signup by setting a password and activating the account.

```typescript
const completeVendorSignup = async (
    params: CompleteVendorSignupParams
): Promise<CompleteVendorSignupResponse> => {
    try {
        const { data, error } = await supabase.functions.invoke("complete-vendor-signup", {
            body: params,
            headers: {
                Authorization: `Bearer ${supabaseAnonKey}`,
                "x-app-client-secret": appClientSecret,
            },
        });

        if (error) throw error;

        return data as CompleteVendorSignupResponse;
    } catch (error) {
        throw error;
    }
};
```

**Usage:**
```typescript
const result = await handleVendors.completeVendorSignup({
    token: onboardingToken,
    password: "securePassword123"
});

if (result.success) {
    console.log("Account activated for user:", result.userId);
    // Now call login to establish session
}
```

See [Vendor Onboarding](/docs/vendor/onboarding) for the complete onboarding flow.

---

### 3. addBusiness

Creates a new business with store and owner vendor. This operation requires super_admin authentication.

```typescript
const addBusiness = async (
    params: AddBusinessParams
): Promise<Response<AddBusinessResponse>> => {
    try {
        const { data, error } = await supabase.functions.invoke("add-business", {
            body: params,
        });

        if (error) throw error;

        return { data, isSuccessful: true, message: "Business added successfully" };
    } catch (error) {
        throw error;
    }
};
```

**Endpoint:** `POST /functions/v1/add-business`

**Request Body (AddBusinessParams):**
```typescript
{
    business_name: string;
    business_address?: string;
    cac_number?: string;
    tax_id?: string;
    cac_document_url?: string;
    store_name: string;
    store_address?: string;
    store_city?: string;
    store_state?: string;
    store_latitude?: number;
    store_longitude?: number;
    store_logo?: { url: string; blur_hash?: string } | null;
    store_cover_photo?: any;
    store_cover_photo_blur_hash?: string;
    vendor_first_name: string;
    vendor_last_name: string;
    vendor_phone_number?: string;
    vendor_profile_photo?: any;
    vendor_profile_photo_hash?: string;
    vendor_email: string;
    working_hours?: WorkingHour[];
    is_development?: boolean;
}
```

**Response (AddBusinessResponse):**
```typescript
{
    success: boolean;
    businessId: string;
    storeId: string;
    vendorUserId: string;
    onboardingToken: string;
}
```

**Usage Example:**
```typescript
const result = await handleVendors.addBusiness({
    business_name: "Example Restaurant",
    business_address: "123 Main St",
    store_name: "Downtown Location",
    store_city: "Lagos",
    store_state: "Lagos",
    vendor_first_name: "John",
    vendor_last_name: "Doe",
    vendor_email: "john@example.com",
    working_hours: [
        {
            day_of_the_week: "monday",
            opening_time: "09:00",
            closing_time: "22:00",
            is_opened: true
        }
    ]
});

if (result.isSuccessful) {
    console.log("Business ID:", result.data.businessId);
    console.log("Store ID:", result.data.storeId);
    console.log("Onboarding Token:", result.data.onboardingToken);
    // Share the onboarding token with the vendor
}
```

**Notes:**
- The `onboardingToken` in the response should be securely shared with the vendor to complete their account setup
- Store logo and cover photo should be uploaded first using the upload API
- Working hours are optional but recommended for proper store operations

---

### 4. addVendor

Adds a vendor to an existing business and store. This operation requires business owner or super_admin authentication.

```typescript
const addVendor = async (
    params: AddVendorParams
): Promise<Response<AddVendorResponse>> => {
    try {
        const { data, error } = await supabase.functions.invoke("add-vendor", {
            body: params,
        });

        if (error) throw error;

        return { data, isSuccessful: true, message: "Vendor added successfully" };
    } catch (error) {
        throw error;
    }
};
```

**Endpoint:** `POST /functions/v1/add-vendor`

**Request Body (AddVendorParams):**
```typescript
{
    vendor_first_name: string;
    vendor_last_name: string;
    vendor_phone_number?: string;
    vendor_profile_photo?: { url: string; blur_hash?: string } | null;
    vendor_profile_photo_hash?: string;
    vendor_email: string;
    vendor_role?: "owner" | "manager" | "member";
    business_id: string;
    store_id: string;
    is_development?: boolean;
}
```

**Response (AddVendorResponse):**
```typescript
{
    success: boolean;
    businessId: string;
    vendorUserId: string;
    onboardingToken: string;
}
```

**Usage Example:**
```typescript
const result = await handleVendors.addVendor({
    vendor_first_name: "Jane",
    vendor_last_name: "Smith",
    vendor_email: "jane@example.com",
    vendor_phone_number: "+2348012345678",
    vendor_role: "manager",
    business_id: "abc123-def456",
    store_id: "store-xyz-789"
});

if (result.isSuccessful) {
    console.log("Vendor User ID:", result.data.vendorUserId);
    console.log("Onboarding Token:", result.data.onboardingToken);
    // Share the onboarding token with the vendor
}
```

**Notes:**
- The `store_id` is required to associate the vendor with a specific store within the business
- The `vendor_role` determines the level of access the vendor will have (owner, manager, or member)
- Vendor profile photo should be uploaded first using the upload API

---

### 5. getAllBusinesses

Retrieves a paginated list of all businesses. This operation requires admin authentication.

```typescript
const getAllBusinesses = async (
    params: GetAllBusinessesParams = {}
): Promise<Response<CompositeTypes<'business_item_response'>[]>> => {
    try {
        const { data, error } = await supabase.rpc("get_all_businesses", {
            p_limit: params.limit || 10,
            p_cursor_business_name: params.cursorBusinessName ?? undefined,
            p_cursor_id: params.cursorId ?? undefined,
            p_start_date: params.startDate ?? undefined,
            p_end_date: params.endDate ?? undefined,
            p_status: params.status ?? undefined,
        });

        if (error) throw error;

        return { data: data || [], isSuccessful: true, message: "Fetched businesses successfully" };
    } catch (error) {
        return {
            data: [],
            isSuccessful: false,
            message: error instanceof Error ? error.message : "Failed to fetch businesses",
        };
    }
};
```

**Usage Example:**
```typescript
// Get first page of businesses
const result = await handleVendors.getAllBusinesses({ limit: 10 });

if (result.isSuccessful) {
    const businesses = result.data;
    businesses.forEach(business => {
        console.log(business.business_name);
        console.log(business.number_of_stores);
        console.log(business.status);
    });
}

// Get next page using cursor
const lastBusiness = businesses[businesses.length - 1];
const nextPage = await handleVendors.getAllBusinesses({
    limit: 10,
    cursorBusinessName: lastBusiness.business_name,
    cursorId: lastBusiness.id
});

// Filter by creation date
const filteredByDate = await handleVendors.getAllBusinesses({
    limit: 10,
    startDate: '2024-01-01T00:00:00Z'
});

// Filter by status
const activeBusinesses = await handleVendors.getAllBusinesses({
    limit: 10,
    status: 'active'
});
```

**Request Parameters (GetAllBusinessesParams):**
```typescript
{
    limit?: number;                // Optional: Results per page (default: 10)
    cursorBusinessName?: string;   // Optional: Cursor for pagination
    cursorId?: string;             // Optional: Cursor for pagination
    startDate?: string;            // Optional: Filter by created_at start (ISO 8601)
    endDate?: string;              // Optional: Filter by created_at end (ISO 8601)
    status?: "active" | "suspended" | "pending_activation"; // Optional: Filter by status
}
```

**Response Type:**
```typescript
{
    data: CompositeTypes<'business_item_response'>[];
    isSuccessful: boolean;
    message: string;
}
```

**Business Item Response:**
```typescript
{
    id: string;
    business_name: string;
    logo: { url: string; blur_hash?: string } | null;
    number_of_stores: number;
    created_at: string;
    updated_at: string;
    status: "active" | "suspended" | "pending_activation";
}
```

---

### 6. searchBusiness

Searches for businesses using fuzzy text search on business name and address. This operation requires admin authentication and uses PostgreSQL's pg_trgm extension.

```typescript
const searchBusiness = async (
    params: SearchBusinessParams
): Promise<Response<CompositeTypes<'business_item_response'>[]>> => {
    try {
        const { data, error } = await supabase.rpc("search_business", {
            p_search_query: params.searchQuery ?? undefined,
            p_limit: params.limit || 10,
            p_cursor_business_name: params.cursorBusinessName ?? undefined,
            p_cursor_id: params.cursorId ?? undefined,
            p_start_date: params.startDate ?? undefined,
            p_end_date: params.endDate ?? undefined,
            p_status: params.status ?? undefined,
        });

        if (error) throw error;

        return { data: data || [], isSuccessful: true, message: "Search completed successfully" };
    } catch (error) {
        return {
            data: [],
            isSuccessful: false,
            message: error instanceof Error ? error.message : "Failed to search businesses",
        };
    }
};
```

**Usage Example:**
```typescript
// Basic search
const result = await handleVendors.searchBusiness({
    searchQuery: 'restaurant'
});

if (result.isSuccessful) {
    const businesses = result.data;
    businesses.forEach(business => {
        console.log(business.business_name);
        console.log(business.business_address);
    });
}

// Search with filters
const filteredSearch = await handleVendors.searchBusiness({
    searchQuery: 'cafe',
    limit: 20,
    status: 'active',
    startDate: '2024-01-01T00:00:00Z'
});

// Paginated search
const lastBusiness = businesses[businesses.length - 1];
const nextPage = await handleVendors.searchBusiness({
    searchQuery: 'restaurant',
    limit: 10,
    cursorBusinessName: lastBusiness.business_name,
    cursorId: lastBusiness.id
});
```

**Request Parameters (SearchBusinessParams):**
```typescript
{
    searchQuery?: string;          // Optional: When omitted/empty, returns all businesses
    limit?: number;                // Optional: Results per page (default: 10)
    cursorBusinessName?: string;   // Optional: Cursor for pagination
    cursorId?: string;             // Optional: Cursor for pagination
    startDate?: string;            // Optional: Filter by created_at start (ISO 8601)
    endDate?: string;              // Optional: Filter by created_at end (ISO 8601)
    status?: "active" | "suspended" | "pending_activation"; // Optional: Filter by status
}
```

**Search Features:**
- Uses PostgreSQL pg_trgm extension for fuzzy text matching
- Searches both `business_name` and `business_address` fields
- Results are ordered by similarity score (most relevant first)
- Supports partial matches and typo tolerance
- A similarity threshold of 0.1 is applied to filter out very poor matches

---

### 7. getAllBusinessSummary

Retrieves summary statistics for all businesses. This operation requires admin authentication.

```typescript
const getAllBusinessSummary = async (
    params: GetAllBusinessSummaryParams = {}
): Promise<Response<CompositeTypes<'business_summary_response'> | null>> => {
    try {
        const { data, error } = await supabase.rpc("get_all_business_summary", {
            p_status: params.status ?? undefined,
            p_start_date: params.startDate ?? undefined,
            p_end_date: params.endDate ?? undefined,
            p_search_query: params.searchQuery ?? undefined,
        });

        if (error) throw error;

        return { data, isSuccessful: true, message: "Fetched business summary successfully" };
    } catch (error) {
        return {
            data: null,
            isSuccessful: false,
            message: error instanceof Error ? error.message : "Failed to fetch business summary",
        };
    }
};
```

**Usage Example:**
```typescript
const result = await handleVendors.getAllBusinessSummary({
    status: 'active',
    startDate: '2024-01-01T00:00:00Z',
    endDate: '2024-01-31T23:59:59Z',
    searchQuery: 'restaurant'
});

if (result.isSuccessful) {
    const summary = result.data;
    console.log('Total businesses:', summary.all_business_count);
    console.log('Active businesses:', summary.active_business_count);
    console.log('Suspended businesses:', summary.suspended_business_count);
    console.log('Pending activation:', summary.pending_activation_business_count);
}
```

**Response Type:**
```typescript
{
    data: {
        all_business_count: number;
        active_business_count: number;
        suspended_business_count: number;
        pending_activation_business_count: number;
    } | null;
    isSuccessful: boolean;
    message: string;
}
```

---

### 8. searchVendors

Searches vendors within a specific store using fuzzy text search. This operation requires admin authentication or vendor access to the same store.

```typescript
const searchVendors = async (
    params: SearchVendorsParams
): Promise<Response<CompositeTypes<'vendor_item_response'>[]>> => {
    try {
        if (!params.storeId) {
            throw new Error("Store ID is required");
        }

        const { data, error } = await supabase.rpc("search_vendors", {
            p_store_id: params.storeId,
            ...(params.searchQuery && { p_search_query: params.searchQuery }),
            p_limit: params.limit || 10,
            ...(params.cursorFirstName && { p_cursor_first_name: params.cursorFirstName }),
            ...(params.cursorId && { p_cursor_id: params.cursorId }),
            ...(params.startDate && { p_start_date: params.startDate }),
            ...(params.endDate && { p_end_date: params.endDate }),
            ...(params.isPendingActivation !== undefined && { p_pending_activation: params.isPendingActivation }),
        });

        if (error) throw error;

        return { data: data || [], isSuccessful: true, message: "Search completed successfully" };
    } catch (error) {
        return {
            data: [],
            isSuccessful: false,
            message: error instanceof Error ? error.message : "Failed to search vendors",
        };
    }
};
```

**Usage Example:**
```typescript
// Basic search within a store
const result = await handleVendors.searchVendors({
    storeId: 'store-uuid-here',
    searchQuery: 'john',
    limit: 10
});

if (result.isSuccessful) {
    const vendors = result.data;
    vendors.forEach(vendor => {
        console.log(vendor.first_name, vendor.last_name, vendor.role);
    });
}

// Filter by pending activation status
const pendingVendors = await handleVendors.searchVendors({
    storeId: 'store-uuid-here',
    isPendingActivation: true,
    limit: 10
});

// Paginated search with cursor
const lastVendor = vendors[vendors.length - 1];
const nextPage = await handleVendors.searchVendors({
    storeId: 'store-uuid-here',
    searchQuery: 'john',
    limit: 10,
    cursorFirstName: lastVendor.first_name,
    cursorId: lastVendor.id
});
```

**Request Parameters (SearchVendorsParams):**
```typescript
{
    storeId: string;             // Required: The store to search within
    searchQuery?: string;        // Optional: Fuzzy search on name/email
    limit?: number;              // Optional: Results per page (default: 10)
    cursorFirstName?: string;    // Optional: Cursor for pagination
    cursorId?: string;           // Optional: Cursor for pagination
    startDate?: string;          // Optional: Filter by created_at start (ISO 8601)
    endDate?: string;            // Optional: Filter by created_at end (ISO 8601)
    isPendingActivation?: boolean; // Optional: Filter by pending activation status
}
```

---

### 9. getAllVendorSummary

Retrieves summary statistics for vendors within a specific store. This operation requires admin authentication or vendor access to the same store.

```typescript
const getAllVendorSummary = async (
    params: GetAllVendorSummaryParams
): Promise<Response<CompositeTypes<'vendor_summary_response'> | null>> => {
    try {
        if (!params.storeId) {
            throw new Error("Store ID is required");
        }

        const { data, error } = await supabase.rpc("get_all_vendor_summary", {
            p_store_id: params.storeId,
            ...(params.isPendingActivation !== undefined && { p_pending_activation: params.isPendingActivation }),
            ...(params.startDate && { p_start_date: params.startDate }),
            ...(params.endDate && { p_end_date: params.endDate }),
            ...(params.searchQuery && { p_search_query: params.searchQuery }),
        });

        if (error) throw error;

        return { data, isSuccessful: true, message: "Fetched vendor summary successfully" };
    } catch (error) {
        return {
            data: null,
            isSuccessful: false,
            message: error instanceof Error ? error.message : "Failed to fetch vendor summary",
        };
    }
};
```

**Usage Example:**
```typescript
const result = await handleVendors.getAllVendorSummary({
    storeId: 'store-uuid-here',
    isPendingActivation: false,
    startDate: '2024-01-01T00:00:00Z',
    endDate: '2024-01-31T23:59:59Z',
    searchQuery: 'manager'
});

if (result.isSuccessful) {
    const summary = result.data;
    console.log('Total vendors:', summary.all_vendor_count);
    console.log('Active vendors:', summary.active_vendor_count);
    console.log('Pending activation:', summary.pending_activation_vendor_count);
}
```

**Request Parameters (GetAllVendorSummaryParams):**
```typescript
{
    storeId: string;             // Required: The store to get summary for
    isPendingActivation?: boolean; // Optional: Filter by pending activation status
    startDate?: string;          // Optional: Filter by created_at start (ISO 8601)
    endDate?: string;            // Optional: Filter by created_at end (ISO 8601)
    searchQuery?: string;        // Optional: Fuzzy search filter
}
```

---

### 10. resendVendorInvite

Resends a vendor invitation email via the `resend-vendor-invite` edge function. This is useful when a vendor needs to receive their onboarding invitation again.

```typescript
const resendVendorInvite = async (params: ResendVendorInviteParams): Promise<Response<any>> => {
    try {
        const { data, error } = await supabase.functions.invoke("resend-vendor-invite", {
            body: params,
        });

        if (error) throw error;

        return { data, isSuccessful: true, message: "Vendor invitation resent successfully" };
    } catch (error) {
        return {
            data: null,
            isSuccessful: false,
            message: error instanceof Error ? error.message : "Failed to resend vendor invitation",
        };
    }
};
```

**Usage Example:**
```typescript
const result = await handleVendors.resendVendorInvite({
    email: "vendor@example.com",
    is_development: false
});

if (result.isSuccessful) {
    console.log("Vendor invitation resent successfully");
} else {
    console.error("Failed to resend invitation:", result.message);
}
```

**Notes:**
- The function requires the app client secret in headers for security
- Use `is_development: true` when testing in development environment
- The vendor will receive a new onboarding link via email

---

### 11. exportVendors

Exports vendors to Excel via edge function. This operation requires admin authentication or vendor access to the same store.

```typescript
const exportVendors = async (params: ExportVendorsParams): Promise<Response<ExportVendorsResponse>> => {
    try {
        if (!params.storeId) {
            throw new Error("Store ID is required");
        }

        const { data, error } = await supabase.functions.invoke("export-vendors", {
            body: params,
        });

        if (error) throw error;

        return { data, isSuccessful: true, message: "Vendors exported successfully" };
    } catch (error) {
        return {
            data: {} as ExportVendorsResponse,
            isSuccessful: false,
            message: error instanceof Error ? error.message : "Failed to export vendors",
        };
    }
};
```

**Usage Example:**
```typescript
const result = await handleVendors.exportVendors({
    storeId: 'store-uuid-here',
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

**Request Parameters (ExportVendorsParams):**
```typescript
{
    storeId: string;             // Required: The store to export vendors from
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

### 12. getStoreById

Retrieves a store by its unique identifier with all columns.

```typescript
const getStoreById = async (id: string): Promise<Response<Tables<'stores'> | null>> => {
    try {
        const { data, error } = await supabase
            .from("stores")
            .select("*")
            .eq("id", id)
            .single();

        if (error) throw error;

        return { data, isSuccessful: true, message: "Store retrieved successfully" };
    } catch (error) {
        throw error;
    }
};
```

**Usage Example:**
```typescript
const result = await handleVendors.getStoreById('store-uuid-here');

if (result.isSuccessful && result.data) {
    console.log('Store Name:', result.data.store_name);
    console.log('Store Address:', result.data.store_address);
    console.log('Business ID:', result.data.business_id);
}
```

**Response Type:**
```typescript
{
    data: Tables<'stores'> | null;
    isSuccessful: boolean;
    message: string;
}
```

---

### 13. getStoresByBusinessId

Retrieves all stores associated with a specific business.

```typescript
const getStoresByBusinessId = async (businessId: string): Promise<Response<Tables<'stores'>[]>> => {
    try {
        const { data, error } = await supabase
            .from("stores")
            .select("*")
            .eq("business_id", businessId);

        if (error) throw error;

        return { data: data || [], isSuccessful: true, message: "Store retrieved successfully" };
    } catch (error) {
        throw error;
    }
};
```

**Usage Example:**
```typescript
const result = await handleVendors.getStoresByBusinessId('business-uuid-here');

if (result.isSuccessful) {
    const stores = result.data;
    stores.forEach(store => {
        console.log('Store:', store.store_name, store.store_address);
    });
}
```

**Response Type:**
```typescript
{
    data: Tables<'stores'>[];
    isSuccessful: boolean;
    message: string;
}
```

---

### 14. getVendorById

Retrieves a vendor by their unique identifier with all columns.

```typescript
const getVendorById = async (id: string): Promise<Response<Tables<'vendors'> | null>> => {
    try {
        const { data, error } = await supabase
            .from("vendors")
            .select("*")
            .eq("id", id)
            .single();

        if (error) throw error;

        return { data, isSuccessful: true, message: "Vendor retrieved successfully" };
    } catch (error) {
        return {
            data: null,
            isSuccessful: false,
            message: error instanceof Error ? error.message : "Failed to fetch vendor",
        };
    }
};
```

**Usage Example:**
```typescript
const result = await handleVendors.getVendorById('vendor-uuid-here');

if (result.isSuccessful && result.data) {
    console.log('Vendor:', result.data.first_name, result.data.last_name);
    console.log('Email:', result.data.vendor_email);
    console.log('Role:', result.data.vendor_role);
}
```

**Response Type:**
```typescript
{
    data: Tables<'vendors'> | null;
    isSuccessful: boolean;
    message: string;
}
```

---

### 15. getBusinessById

Retrieves a business by its unique identifier with all columns.

```typescript
const getBusinessById = async (id: string): Promise<Response<Tables<'businesses'> | null>> => {
    try {
        const { data, error } = await supabase
            .from("businesses")
            .select("*")
            .eq("id", id)
            .single();

        if (error) throw error;

        return { data, isSuccessful: true, message: "Business retrieved successfully" };
    } catch (error) {
        return {
            data: null,
            isSuccessful: false,
            message: error instanceof Error ? error.message : "Failed to fetch business",
        };
    }
};
```

**Usage Example:**
```typescript
const result = await handleVendors.getBusinessById('business-uuid-here');

if (result.isSuccessful && result.data) {
    console.log('Business:', result.data.business_name);
    console.log('Status:', result.data.status);
    console.log('CAC Number:', result.data.cac_number);
}
```

**Response Type:**
```typescript
{
    data: Tables<'businesses'> | null;
    isSuccessful: boolean;
    message: string;
}
```

---

## Export

```typescript
export const handleVendors = {
    getOnboardingVendor,
    completeVendorSignup,
    addBusiness,
    addVendor,
    getAllBusinesses,
    searchBusiness,
    getAllBusinessSummary,
    searchVendors,
    getAllVendorSummary,
    resendVendorInvite,
    exportVendors,
    getStoreById,
    getStoresByBusinessId,
    getVendorById,
    getBusinessById,
};
```

---

## Security Notes

- **Admin Functions**: `getAllBusinesses`, `searchBusiness`, `getAllBusinessSummary`, `searchVendors`, `getAllVendorSummary`, and `exportVendors` require admin authentication
- **Business Management**: `addBusiness` requires `super_admin` role; `addVendor` requires business owner or `super_admin` role
- **Onboarding Functions**: `getOnboardingVendor` and `completeVendorSignup` are public but require the app client secret header
- **Export Functions**: Require appropriate authentication and store access permissions
- All functions use `security definer` database functions where applicable and perform role checks internally