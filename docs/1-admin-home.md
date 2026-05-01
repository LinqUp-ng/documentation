# Admin Home Screen
*Setting up the ecosystem and internal team controls.*

### 1. Admin Operations
- [x] **Recent Activities:** `GET /activities` - Fetches cursor-paginated system activities. The system automatically filters results based on the caller's context:
    - **Admins:** See all activities across the platform.
    - **Vendors:** See activities related to their logged-in `business_id`.
    - **Users:** See their own activities linked to their `user_id`.
- [x] **Dashboard Analytics:** `GET /admin/analytics` - Fetches platform analytics including transaction data, subscription data, and percentage differences. Only accessible by admins.
- [x] **Recent Transactions:** `GET /transactions` - Fetches cursor-paginated ledger transactions. The system automatically filters results based on the caller's context:
    - **Admins:** See all transactions across the platform.
    - **Vendors:** See transactions for their store wallet (store_id required).
    - **Users:** See transactions for their personal wallet.
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

#### Implementation: `api/handleRecentTransactions.ts`

```typescript
import { supabase } from "@/lib/supabase";
import { Response } from "@/types/api";

export interface GetRecentTransactionsParams {
  limit?: number;
  store_id?: string;
  cursor_id?: string;
  cursor_timestamp?: string;
}

export interface TransactionItemResponse {
  id: number;
  wallet_id: string;
  entry_type: 'credit' | 'debit';
  amount: number;
  balance_after: number;
  reference: string;
  description: string | null;
  transaction_id: string | null;
  created_at: string;
  wallet: {
    id: string;
    owner_type: 'user' | 'business' | 'store' | 'platform';
    balance: number;
    is_active: boolean;
  } | null;
  profile: {
    id: string;
    name: string;
    profile_photo: string | null;
  } | null;
  business: {
    id: string;
    business_name: string;
  } | null;
  store: {
    id: string;
    store_name: string;
    cover_photo: string | null;
    cover_photo_blur_hash: string | null;
  } | null;
}

/**
 * Fetch recent transactions with cursor-based pagination.
 * The system automatically filters results based on the caller's context:
 * - Admins: See all transactions across the platform
 * - Vendors: See transactions for their store wallet (store_id required)
 * - Users: See transactions for their personal wallet
 */
const getRecentTransactions = async (
  params: GetRecentTransactionsParams = {}
): Promise<Response<TransactionItemResponse[]>> => {
  try {
    const { 
      limit = 10, 
      store_id, 
      cursor_id, 
      cursor_timestamp 
    } = params;

    const { data, error } = await (supabase.rpc as any)('get_recent_transactions', {
      p_limit: limit,
      ...(store_id && { p_store_id: store_id }),
      ...(cursor_id && { p_cursor_id: cursor_id }),
      ...(cursor_timestamp && { p_cursor_timestamp: cursor_timestamp }),
    });

    if (error) throw error;

    return {
      isSuccessful: true,
      message: "Recent transactions fetched successfully",
      data: (data as TransactionItemResponse[]) || []
    };
  } catch (error) {
    throw error;
  }
};

export default {
  getRecentTransactions,
};
```

#### Implementation: `api/handlePlatformAnalytics.ts`

```typescript
import { supabase } from "@/lib/supabase";
import { Response } from "@/types/api";

export interface GetPlatformAnalyticsParams {
  start_date?: string;
  end_date?: string;
}

export interface PlatformAnalyticsResponse {
  start_date: string;
  end_date: string;
  transactions: {
    label: string;
    value: number;
  }[];
  subscriptions: {
    label: string;
    value: number;
  }[];
  percentage_difference: number | null;
}

/**
 * Fetch platform analytics for admin dashboard.
 * Returns transaction and subscription data with percentage differences.
 * Only accessible by admin users.
 */
const getPlatformAnalytics = async (
  params: GetPlatformAnalyticsParams = {}
): Promise<Response<PlatformAnalyticsResponse>> => {
  try {
    const { 
      start_date, 
      end_date 
    } = params;

    const { data, error } = await (supabase.rpc as any)('get_platform_analytics', {
      p_start_date: start_date,
      p_end_date: end_date,
    });

    if (error) throw error;

    return {
      isSuccessful: true,
      message: "Platform analytics fetched successfully",
      data: data as PlatformAnalyticsResponse
    };
  } catch (error) {
    throw error;
  }
};

/**
 * Fetch raw platform ledger data for debugging.
 * Only accessible by admin users.
 */
const getPlatformLedgerDebug = async (
  params: GetPlatformAnalyticsParams = {}
): Promise<Response<any[]>> => {
  try {
    const { 
      start_date, 
      end_date 
    } = params;

    const { data, error } = await (supabase.rpc as any)('get_platform_ledger_debug', {
      p_start_date: start_date,
      p_end_date: end_date,
    });

    if (error) throw error;

    return {
      isSuccessful: true,
      message: "Platform ledger debug data fetched successfully",
      data: data || []
    };
  } catch (error) {
    throw error;
  }
};

export default {
  getPlatformAnalytics,
  getPlatformLedgerDebug,
};
```

### 2. Vendor Onboarding & Teams
- [ ] **Onboarding:** `POST /admin/vendors` (Invite business), `GET /admin/vendors`.
- [ ] **Vendor Team:** `GET /vendor/team`, `POST /vendor/team` (Invite staff with assigned roles).
- [ ] **Permissions:** `GET /vendor/team/permissions` - Fetch role-based access control (RBAC).

> [!NOTE]
> The backend is business-aware. Most vendor-facing endpoints automatically scoped to the user's `business_id` derived from their JWT claims, ensuring data isolation between different vendors.

