# Phase 1: Admin & Vendor Management
*Setting up the ecosystem and internal team controls.*

### 1. Admin Operations
- [x] **Recent Activities:** `GET /activities` - Fetches cursor-paginated system activities. The system automatically filters results based on the caller's context:
    - **Admins:** See all activities across the platform.
    - **Vendors:** See activities related to their logged-in `business_id`.
    - **Users:** See their own activities linked to their `user_id`.
- [x] **Dashboard Analytics:** `GET /admin/analytics` - Fetches platform analytics including transaction data, subscription data, and percentage differences. Only accessible by admins.
- [ ] **KYC & Moderation:** `GET /admin/kyc`, `POST /admin/kyc/:id/review`, `GET /admin/reports`.
- [ ] **Team:** `GET /admin/team`, `POST /admin/team` (Add with system-generated password).

#### Implementation: `api/handleActivities.ts`

```typescript
import { supabase } from "@/lib/supabase";
import { Response } from "@/types/api";
import { CompositeTypes, Database } from "@/types/database.types";

// Export enums from database
export type ActivityType = Database['public']['Enums']['activity_type'];
export type UserType = Database['public']['Enums']['user_type'];

export interface GetActivitiesParams {
  limit?: number;
  store_id?: string;
  order_id?: string;
  cursor_id?: string;
  cursor_timestamp?: string;
}

/**
 * Fetch activities with cursor-based pagination.
 * The system automatically filters results based on the caller's context:
 * - Admins: See all activities across the platform
 * - Vendors: See activities related to their logged-in business_id
 * - Users: See their own activities linked to their user_id
 * 
 * @param params - Pagination and filter parameters
 * @returns Paginated list of activities
 */
const getActivities = async (
  params: GetActivitiesParams = {}
): Promise<Response<Activity[]>> => {
  try {
    const { 
      limit = 10, 
      store_id, 
      order_id, 
      cursor_id, 
      cursor_timestamp 
    } = params;

    const { data, error } = await supabase.rpc('get_activities', {
      p_limit: limit,
      p_store_id: store_id || null,
      p_order_id: order_id || null,
      p_cursor_id: cursor_id || null,
      p_cursor_timestamp: cursor_timestamp || null
    });

    if (error) throw error;

    return {
      isSuccessful: true,
      message: "Activities fetched successfully",
      data: data || []
    };
  } catch (error) {
    throw error;
  }
};

export default {
  getActivities,
};
```

### 2. Vendor Onboarding & Teams
- [ ] **Onboarding:** `POST /admin/vendors` (Invite business), `GET /admin/vendors`.
- [ ] **Vendor Team:** `GET /vendor/team`, `POST /vendor/team` (Invite staff with assigned roles).
- [ ] **Permissions:** `GET /vendor/team/permissions` - Fetch role-based access control (RBAC).

> [!NOTE]
> The backend is business-aware. Most vendor-facing endpoints automatically scoped to the user's `business_id` derived from their JWT claims, ensuring data isolation between different vendors.

