# Vendor Management

This document covers vendor-related operations for administrators.

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

### Cursor Pagination

- Results are sorted by `business_name` then `id` in ascending order
- To paginate, pass the `business_name` and `id` of the last item from the previous page as `cursorBusinessName` and `cursorId`
- The `limit` parameter controls the number of results per page (default: 10)

## Get Business Summary

Retrieves summary statistics for all businesses. This operation requires admin authentication.

### Usage Example

```typescript
import { handleVendors } from '@/api/handleVendors';
import { CompositeTypes } from '@/types/database.types';

const result = await handleVendors.getAllBusinessSummary();

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

### Notes for Admin Functions

- Both `getAllBusinesses` and `getAllBusinessSummary` require admin authentication
- These functions use `security definer` and perform admin-only checks internally
- They return type-safe responses using CompositeTypes from database.types.ts
- Handle errors gracefully by checking `isSuccessful` before accessing data