# Vendor Management

This document covers vendor-related operations for administrators.

## API Implementation

```typescript
@/Users/ovieokomite-iffie/Documents/mobile apps/linqUp/api/handleVendors.ts:1-257
import { Enums, supabase } from "@/lib/supabase";
import { Response } from "@/types/api";
import { CompositeTypes, Database } from "@/types/database.types";
import { UploadWithBlurhashResponse } from "./handleUpload";

const supabaseAnonKey = process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY || "";
const appClientSecret = process.env.EXPO_PUBLIC_APP_CLIENT_SECRET || "";

export type DayOfWeek = Database["public"]["Enums"]["day_of_the_week"];

export interface WorkingHour {
    day_of_the_week: DayOfWeek;
    opening_time: string;
    closing_time: string;
    is_opened?: boolean;
}

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

export interface CompleteAdminSignupResponse {
    success: boolean;
    message: string;
    userId: string;
    session?: any;
    user?: any;
}

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
}

/**
 * Get onboarding vendor details via edge function
 * Public function - no auth required
 */
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

/**
 * Complete vendor signup by setting password and activating account
 */
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
        console.log("🚀 ~ completeVendorSignup ~ data:", data)
        console.log("🚀 ~ completeVendorSignup ~ error:", JSON.stringify(error))
    
        if (error) {
            throw error
        }
    
        return data as CompleteVendorSignupResponse;
    } catch (error) {
        throw error;
    }
};

/**
 * Add a new business with store and owner vendor via edge function
 * Requires super_admin authentication
 */
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

/**
 * Add a vendor to an existing business via edge function
 * Requires business owner or super_admin authentication
 */
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

/**
 * Get all businesses with cursor pagination
 * Admin only - requires admin authentication
 */
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

/**
 * Search businesses with fuzzy text search using pg_trgm
 * Admin only - requires admin authentication
 */
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

/**
 * Get business summary statistics
 * Admin only - requires admin authentication
 */
const getAllBusinessSummary = async (params: GetAllBusinessSummaryParams = {}): Promise<
    Response<CompositeTypes<'business_summary_response'> | null>
> => {
    try {
        const { data, error } = await supabase.rpc("get_all_business_summary", {
            p_status: params.status ?? undefined,
            p_start_date: params.startDate ?? undefined,
            p_end_date: params.endDate ?? undefined,
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

/**
 * Resend vendor invitation email via edge function
 */
const resendVendorInvite = async (params: ResendVendorInviteParams): Promise<Response<any>> => {
    try {
        const { data, error } = await supabase.functions.invoke("resend-vendor-invite", {
            body: params,
            headers: {
                Authorization: `Bearer ${supabaseAnonKey}`,
                "x-app-client-secret": appClientSecret,
            },
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

export const handleVendors = {
    getOnboardingVendor,
    completeVendorSignup,
    addBusiness,
    addVendor,
    getAllBusinesses,
    searchBusiness,
    getAllBusinessSummary,
    resendVendorInvite,
};
```

## Add Business

Creates a new business with store and owner vendor. This operation requires super_admin authentication.

### Endpoint
`POST /functions/v1/add-business`

### Request Body (AddBusinessParams)

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
  store_logo?: {
    url: string;
    blur_hash?: string;
  } | null;
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

### WorkingHour Format

```typescript
{
  day_of_week: DayOfWeek; // enum: monday, tuesday, wednesday, thursday, friday, saturday, sunday
  opening_time: string;   // format: "HH:mm"
  closing_time: string;   // format: "HH:mm"
  is_opened?: boolean;
}
```

### Response (AddBusinessResponse)

```typescript
{
  success: boolean;
  businessId: string;
  storeId: string;
  vendorUserId: string;
  onboardingToken: string;
}
```

### Usage Example

```typescript
import { handleVendors } from '@/api/handleVendors';

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
      day_of_week: "monday",
      opening_time: "09:00",
      closing_time: "22:00",
      is_opened: true
    }
  ]
});

console.log(result.data.onboardingToken); // Share this with the vendor for onboarding
```

### Notes

- The `onboardingToken` in the response should be securely shared with the vendor to complete their account setup
- The vendor will use this token to retrieve their onboarding details and complete signup
- Store logo and cover photo should be uploaded first using the upload API
- Working hours are optional but recommended for proper store operations

## Get All Businesses

Retrieves a paginated list of all businesses. This operation requires admin authentication.

### Usage Example

```typescript
import { handleVendors } from '@/api/handleVendors';
import { Database } from '@/types/database.types';

// Get first page of businesses
const result = await handleVendors.getAllBusinesses({
    limit: 10
});

if (result.isSuccessful) {
    const businesses = result.data; // Database["public"]["CompositeTypes"]["business_item_response"][]
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

// Filter by creation date (businesses created after a specific time)
const filteredByDate = await handleVendors.getAllBusinesses({
    limit: 10,
    startDate: '2024-01-01T00:00:00Z'
});

// Filter by creation date range
const filteredByDateRange = await handleVendors.getAllBusinesses({
    limit: 10,
    startDate: '2024-01-01T00:00:00Z',
    endDate: '2024-01-31T23:59:59Z'
});

// Filter by status
const activeBusinesses = await handleVendors.getAllBusinesses({
    limit: 10,
    status: 'active'
});

// Combine filters
const filteredResults = await handleVendors.getAllBusinesses({
    limit: 10,
    startDate: '2024-01-01T00:00:00Z',
    status: 'active'
});
```

### Response Type

```typescript
{
    data: Database["public"]["CompositeTypes"]["business_item_response"][];
    isSuccessful: boolean;
    message: string;
}
```

### Business Item Response

```typescript
{
    id: string;
    business_name: string;
    logo: Database["public"]["CompositeTypes"]["image_type"] | null;
    number_of_stores: number;
    created_at: string;
    updated_at: string;
    status: "active" | "suspended" | "pending_activation";
}
```

### Filter Parameters

- `startDate` (optional): Filter to return businesses created on/after this timestamp (ISO 8601)
- `endDate` (optional): Filter to return businesses created on/before this timestamp (ISO 8601)
- `status` (optional): Filter by business status (`"active" | "suspended" | "pending_activation"`)

### Cursor Pagination

- Results are sorted by `business_name` then `id` in ascending order
- To paginate, pass the `business_name` and `id` of the last item from the previous page as `cursorBusinessName` and `cursorId`
- The `limit` parameter controls the number of results per page (default: 10)
- Filters can be combined with cursor pagination for efficient data retrieval

## Get Business Summary

Retrieves summary statistics for all businesses. This operation requires admin authentication.

### Usage Example

```typescript
import { handleVendors } from '@/api/handleVendors';
import { CompositeTypes } from '@/types/database.types';

const result = await handleVendors.getAllBusinessSummary({
    status: 'active',
    startDate: '2024-01-01T00:00:00Z',
    endDate: '2024-01-31T23:59:59Z'
});

if (result.isSuccessful) {
    const summary = result.data; // CompositeTypes<'business_summary_response'>
    console.log('Total businesses:', summary.all_business_count);
    console.log('Active businesses:', summary.active_business_count);
    console.log('Suspended businesses:', summary.suspended_business_count);
    console.log('Pending activation:', summary.pending_activation_business_count);
}
```

### Response Type

```typescript
{
    data: CompositeTypes<'business_summary_response'> | null;
    isSuccessful: boolean;
    message: string;
}
```

### Business Summary Response

```typescript
{
    all_business_count: number;
    active_business_count: number;
    suspended_business_count: number;
    pending_activation_business_count: number;
}
```

## Resend Vendor Invite

Resends a vendor invitation email via the `resend-vendor-invite` edge function. This is useful when a vendor needs to receive their onboarding invitation again.

### Request Body (ResendVendorInviteParams)

```typescript
{
  email: string;
  is_development?: boolean;
}
```

### Usage Example

```typescript
import { handleVendors } from '@/api/handleVendors';

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

### Notes

- The function requires the app client secret in headers for security
- Use `is_development: true` when testing in development environment
- The vendor will receive a new onboarding link via email

## Search Businesses

Searches for businesses using fuzzy text search on business name and address. This operation requires admin authentication and uses PostgreSQL's pg_trgm extension for improved search capabilities.

### Usage Example

```typescript
import { handleVendors } from '@/api/handleVendors';
import { Database } from '@/types/database.types';

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

### Request Parameters (SearchBusinessParams)

```typescript
{
    searchQuery?: string;          // Optional: When omitted/empty, returns all businesses (still supports status/date/pagination)
    limit?: number;                // Optional: Results per page (default: 10)
    cursorBusinessName?: string;   // Optional: Cursor for pagination
    cursorId?: string;             // Optional: Cursor for pagination
    startDate?: string;            // Optional: Filter by created_at start (ISO 8601)
    endDate?: string;              // Optional: Filter by created_at end (ISO 8601)
    status?: Database["public"]["Enums"]["business_status_enum"]; // Optional: Filter by status
}
```

### Response Type

```typescript
{
    data: Database["public"]["CompositeTypes"]["business_item_response"][];
    isSuccessful: boolean;
    message: string;
}
```

### Search Features

- Uses PostgreSQL pg_trgm extension for fuzzy text matching
- Searches both `business_name` and `business_address` fields
- Results are ordered by similarity score (most relevant first), then by business name and id
- Supports partial matches and typo tolerance
- Can be combined with date and status filters
- Supports cursor pagination for large result sets

### Notes

- The search uses similarity scoring to rank results by relevance
- A similarity threshold of 0.1 is applied to filter out very poor matches
- The `%` operator (trigram matching) provides additional fuzzy matching capabilities
- For best performance, ensure the pg_trgm extension is enabled in your PostgreSQL database

### Notes for Admin Functions

- `getAllBusinesses`, `searchBusiness`, and `getAllBusinessSummary` require admin authentication
- These functions use `security definer` and perform admin-only checks internally
- They return type-safe responses using CompositeTypes from database.types.ts
- Handle errors gracefully by checking `isSuccessful` before accessing data
