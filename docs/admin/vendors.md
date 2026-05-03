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