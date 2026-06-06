# Order Handling Functions

This document describes the utility functions available for handling orders in the vendor context.

## Source File: `api/handleOrders.ts`

```typescript
import { MenuData } from '@/database/services/handleLocalMenu';
import { Enums, supabase } from '@/lib/supabase';
import { Response } from '@/types/api';
import { CompositeTypes, Tables, TablesInsert, TablesUpdate } from '@/types/database.types';

/** Response type for redeem_booking_code RPC */
export interface RedeemBookingCodeResult {
    success: boolean;
    error?: string;
    booking_code?: {
        id: string;
        code: string;
        order_id: string;
        profile_id: string;
        status: string;
    };
    [key: string]: unknown;
}


/**
 * Insert a new order
 * @param orderData - The order data to insert
 * @returns The inserted order or error
 */
const insertOrder = async (
  orderData: TablesInsert<'orders'>
): Promise<Response<Tables<'orders'> >> => {
    try {
        const { data, error } = await supabase
            .from('orders')
            .insert(orderData)
            .select()
            .single();

        if (error) throw error;

        return {
            isSuccessful: true,
            message: 'Order inserted successfully',
            data: data,
        };
    } catch (error) {
        console.error('Error inserting order:', error);
        throw error;
    }
};

/**
 * Update an existing order
 * @param orderId - The ID of the order to update
 * @param orderData - The order data to update
 * @returns The updated order or error
 */
const updateOrder = async (
  orderId: string,
  orderData: TablesUpdate<'orders'>
): Promise<Response<Tables<'orders'> >> => {
    try {
        const { data, error } = await supabase
            .from('orders')
            .update(orderData)
            .eq('id', orderId)
            .select()
            .single();

        if (error) throw error;

        return {
            isSuccessful: true,
            message: 'Order updated successfully',
            data: data,
        };
    } catch (error) {
        console.error('Error updating order:', error);
        throw error;
    }
};

/**
 * Delete an order
 * @param orderId - The ID of the order to delete
 * @returns Success or error
 */
const deleteOrder = async (
  orderId: string
): Promise<Response<void>> => {
    try {
        const { error } = await supabase
            .from('orders')
            .delete()
            .eq('id', orderId);

        if (error) throw error;

        return {
            isSuccessful: true,
            message: 'Order deleted successfully',
            data: undefined,
        };
    } catch (error) {
        console.error('Error deleting order:', error);
        throw error;
    }
};

/**
 * Get orders by match ID
 * @param matchId - The ID of the match to get orders for
 * @returns The orders or error
 */
const getOrdersByMatchId = async (
  matchId: string
): Promise<Response<(Tables<'orders'> & { order_items: Tables<'order_items'>[] })[]>> => {
    try {
        const { data, error } = await supabase
            .from('orders')
            .select(`
                *,
                order_items (*)
            `)
            .eq('match_id', matchId);

        if (error) throw error;

        return {
            isSuccessful: true,
            message: 'Orders retrieved successfully',
            data: data || [],
        };
    } catch (error) {
        console.error('Error retrieving orders by match ID:', error);
        throw error;
    }
};

/**
 * Get orders by store ID
 * @param storeId - The ID of the store to get orders for
 * @returns The orders or error
 */
const getOrdersByStoreId = async (
  storeId: string
): Promise<Response<Tables<'orders'>[] >> => {
    try {
        const { data, error } = await supabase
            .from('orders')
            .select()
            .eq('store_id', storeId);

        if (error) throw error;

        return {
            isSuccessful: true,
            message: 'Orders retrieved successfully',
            data: data || [],
        };
    } catch (error) {
        console.error('Error retrieving orders by store ID:', error);
        throw error;
    }
};

/**
 * Get orders by profile ID
 * @param profileId - The ID of the profile to get orders for
 * @returns The orders or error
 */
const getOrdersByProfileId = async (
  profileId: string
): Promise<Response<Tables<'orders'>[] >> => {
    try {
        const { data, error } = await supabase
            .from('orders')
            .select()
            .eq('profile_id', profileId);

        if (error) throw error;

        return {
            isSuccessful: true,
            message: 'Orders retrieved successfully',
            data: data || [],
        };
    } catch (error) {
        console.error('Error retrieving orders by profile ID:', error);
        throw error;
    }
};

/**
 * Create a new order using the database function
 * @param matchId - The ID of the match
 * @param storeId - The ID of the store
 * @param items - Array of order items
 * @param sessionDate - Optional session date (defaults to current date)
 * @returns The created order details or error
 */
const createOrder = async ({matchId, storeId, items, sessionDate}: {
    matchId: string,
    storeId: string,
    items: MenuData[], // TODO: Replace with specific type when available
    sessionDate?: string
}): Promise<Response<CompositeTypes<"create_order_response">>> => {
    try {
        const { data, error } = await supabase
            .rpc('create_order', {
                p_match_id: matchId,
                p_store_id: storeId,
                p_items: items,
                p_session_date: sessionDate
            });

        if (error) throw error;

        return {
            isSuccessful: true,
            message: 'Order created successfully',
            data: data,
        };
    } catch (error) {
        console.error('Error creating order:', error);
        throw error;
    }
};

/**
 * Search match orders with pagination and filters
 * @param params - Search parameters
 * @returns The matching orders or error
 */
const searchMatchOrders = async (
    params: {
        searchQuery?: string;
        limit?: number;
        cursorUpdatedAt?: string; // ISO string
        cursorId?: string;
        startDate?: string; // ISO string
        endDate?: string; // ISO string
        status?: Enums<"match_order_status">;
        storeId?: string;
        matchId?: string;
    }
): Promise<Response<CompositeTypes<"match_order_item_response">[]>> => {
    try {
        const { data, error } = await supabase
            .rpc('search_match_orders', {
                ...(params.searchQuery !== undefined && { p_search_query: params.searchQuery }),
                ...(params.limit !== undefined && { p_limit: params.limit }),
                ...(params.cursorUpdatedAt !== undefined && { p_cursor_updated_at: params.cursorUpdatedAt }),
                ...(params.cursorId !== undefined && { p_cursor_id: params.cursorId }),
                ...(params.startDate !== undefined && { p_start_date: params.startDate }),
                ...(params.endDate !== undefined && { p_end_date: params.endDate }),
                ...(params.status !== undefined && { p_status: params.status }),
                ...(params.storeId !== undefined && { p_store_id: params.storeId }),
                ...(params.matchId !== undefined && { p_match_id: params.matchId }),
            });

        if (error) throw error;

        return {
            isSuccessful: true,
            message: 'Match orders retrieved successfully',
            data: data || [],
        };
    } catch (error) {
        console.error('Error searching match orders:', error);
        throw error;
    }
};

/**
 * Get summary of match orders with counts per status
 * @param params - Summary parameters
 * @returns The summary or error
 */
const getMatchOrdersSummary = async (
    params: {
        startDate?: string; // ISO string
        endDate?: string; // ISO string
        status?: Enums<"match_order_status">;
        storeId?: string;
        matchId?: string;
        searchQuery?: string;
    }
): Promise<Response<CompositeTypes<"match_order_summary_response">>> => {
    try {
        const { data, error } = await supabase
            .rpc('get_match_orders_summary', {
                ...(params.startDate !== undefined && { p_start_date: params.startDate }),
                ...(params.endDate !== undefined && { p_end_date: params.endDate }),
                ...(params.status !== undefined && { p_status: params.status }),
                ...(params.storeId !== undefined && { p_store_id: params.storeId }),
                ...(params.matchId !== undefined && { p_match_id: params.matchId }),
                ...(params.searchQuery !== undefined && { p_search_query: params.searchQuery }),
            });

        if (error) throw error;

        return {
            isSuccessful: true,
            message: 'Match orders summary retrieved successfully',
            data: data || null,
        };
    } catch (error) {
        console.error('Error retrieving match orders summary:', error);
        throw error;
    }
};

type MatchOrderResponse = {
    match_order_id: string,
    order_number: string,
    match_id: string,
    store_id: string,
    status: Enums<"match_order_status">,
    total_price: number,
    session_date: string,
    customers: CompositeTypes<"match_order_detail_customer_response">[],
    created_at: string,
    updated_at: string
}

/**
 * Get details for a single match order
 * @param matchOrderId - The ID of the match order
 * @returns The match order details or error
 */
const getMatchOrderDetails = async (
    matchOrderId: string
): Promise<Response<MatchOrderResponse[]>> => {
    try {
        const { data, error } = await supabase
            .rpc('get_match_order_details', {
                p_match_order_id: matchOrderId,
            });

        if (error) throw error;

        return {
            isSuccessful: true,
            message: 'Match order details retrieved successfully',
            data: data || null,
        };
    } catch (error) {
        console.error('Error retrieving match order details:', error);
        throw error;
    }
};

/**
 * Redeem booking code via edge function
 * Requires vendor authentication
 * @param code - Booking code to redeem
 * @param orderId - Order ID associated with the booking code
 * @returns Redemption result
 */
const redeemBookingCode = async (
    code: string,
    orderId: string
): Promise<Response<RedeemBookingCodeResult>> => {
    try {
        const { data, error } = await supabase.functions.invoke("redeem-booking-code", {
            body: { code, order_id: orderId },
        });

        if (error) {
            const errorBody = await error.context.json(); // Try to peek into the body
            console.log("🚀 ~ redeemBookingCode ~ errorBody:", errorBody)

            throw new Error(errorBody?.error)
        }

        const result = data as RedeemBookingCodeResult;
        if (!result?.success) {
            throw new Error(result?.error || "Failed to redeem booking code");
        }

        return { data: result, isSuccessful: true, message: "Booking code redeemed successfully" };
    } catch (error) {
        throw error;
    }
};

export default {
    insertOrder,
    updateOrder,
    deleteOrder,
    createOrder,
    searchMatchOrders, // vendor | admin
    getOrdersByMatchId, // vendor | admin
    getOrdersByStoreId, // vendor | admin
    getOrdersByProfileId, // user
    getMatchOrdersSummary, // vendor | admin
    getMatchOrderDetails, // vendor | admin
    redeemBookingCode, // vendor
};
```

---

## Function Explanations

### insertOrder

**Purpose**: Inserts a new order into the `orders` table.

**Usage**: Used when a user creates an order for a store within a match. This is the basic order creation function that inserts order data directly into the database.

**Key Points**:
- Accepts `TablesInsert<'orders'>` type for order data
- Returns the inserted order record
- Throws error if insertion fails

---

### updateOrder

**Purpose**: Updates an existing order's information.

**Usage**: Used to modify order details such as status, items, or other editable fields before the order is finalized.

**Key Points**:
- Requires the order ID and update data
- Returns the updated order record
- Throws error if update fails

---

### deleteOrder

**Purpose**: Deletes an order from the database.

**Usage**: Used to remove orders, typically when a user cancels before fulfillment or during admin cleanup operations.

**Key Points**:
- Requires the order ID
- Returns success status
- Throws error if deletion fails

---

### getOrdersByMatchId

**Purpose**: Retrieves all orders belonging to a specific match, including their order items.

**Usage**: Used by vendors and admins to view all orders placed for a particular match event. This helps in consolidating orders from multiple users for the same match.

**Key Points**:
- Includes nested `order_items` via Supabase relationship query
- Returns array of orders with items
- Used by: vendor, admin

---

### getOrdersByStoreId

**Purpose**: Retrieves all orders for a specific store.

**Usage**: Used by vendors and admins to view all orders placed at a particular store. This helps vendors track incoming orders for their establishment.

**Key Points**:
- Filters orders by `store_id`
- Returns array of orders
- Used by: vendor, admin

---

### getOrdersByProfileId

**Purpose**: Retrieves all orders placed by a specific user profile.

**Usage**: Used by users to view their own order history. Shows all orders a particular user has placed across different matches and stores.

**Key Points**:
- Filters orders by `profile_id`
- Returns array of orders
- Used by: user

---

### createOrder

**Purpose**: Creates a new order using the `create_order` database RPC function.

**Usage**: The primary function for creating orders in the system. Unlike `insertOrder`, this uses a database function that likely handles additional business logic such as order validation, match verification, and item processing.

**Key Points**:
- Accepts match ID, store ID, items array, and optional session date
- Uses `MenuData` type for order items
- Returns `create_order_response` composite type
- Used by: vendor, admin

---

### searchMatchOrders

**Purpose**: Searches match orders with pagination, filters, and fuzzy search capabilities.

**Usage**: The main search function for finding match orders. Supports filtering by date range, status, store, match, and text search on customer names or order items. Supports cursor-based pagination for large result sets.

**Key Points**:
- Supports fuzzy search via `searchQuery`
- Cursor-based pagination using `cursorUpdatedAt` and `cursorId`
- Filters: date range (`startDate`/`endDate`), `status`, `storeId`, `matchId`
- Returns `match_order_item_response` composite types
- Used by: vendor, admin

---

### getMatchOrdersSummary

**Purpose**: Retrieves a summary of match orders with counts aggregated by status.

**Usage**: Used for dashboard displays and analytics. Provides a quick overview of order status distribution (e.g., how many orders are pending, confirmed, fulfilled, etc.) within a given date range or filter set.

**Key Points**:
- Returns `match_order_summary_response` composite type
- Supports the same filters as `searchMatchOrders` (date range, status, store, match, search)
- Used by: vendor, admin

---

### getMatchOrderDetails

**Purpose**: Retrieves detailed information for a single match order, including customer data and order items.

**Usage**: Used when viewing the details of a specific match order. Returns comprehensive information including all customers in the order, their order statuses, escrow status, booking codes, and individual order items with menu group details.

**Key Points**:
- Uses the `get_match_order_details` RPC function
- Returns `MatchOrderResponse` type containing match order info and nested customer details
- Customer details include: profile info, order status, escrow/verification status, booking code, and order items
- Used by: vendor, admin

---

### redeemBookingCode

**Purpose**: Redeems a booking code via an edge function.

**Usage**: Used by vendors to verify and redeem a booking code when a customer arrives for their scheduled linq up. This function calls the `redeem-booking-code` edge function which validates the booking code and marks the associated escrow as released.

**Redemption Conditions**:
- Both parties in the match order must have been paid
- Both parties must have scheduled a linq up date
- The booking code cannot be redeemed before the scheduled date

**Key Points**:
- Requires vendor authentication
- Invokes the `redeem-booking-code` edge function
- Returns `RedeemBookingCodeResult` with booking code details
- Throws error if redemption fails or conditions are not met
- Used by: vendor

---

## Export

```typescript
export default {
    insertOrder,
    updateOrder,
    deleteOrder,
    createOrder,
    searchMatchOrders, // vendor | admin
    getOrdersByMatchId, // vendor | admin
    getOrdersByStoreId, // vendor | admin
    getOrdersByProfileId, // user
    getMatchOrdersSummary, // vendor | admin
    getMatchOrderDetails, // vendor | admin
    redeemBookingCode, // vendor
};
```
