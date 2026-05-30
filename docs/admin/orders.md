# Order Handling Functions

This document describes the functions available for handling orders in the admin context.

## searchMatchOrders

Search for match orders with pagination and filters.

### Parameters

- `searchQuery` (string, optional): The search query to match against customer names or order item names.
- `limit` (number, optional): The maximum number of results to return (default: 10).
- `cursorUpdatedAt` (string, optional): The cursor for pagination based on the updated_at timestamp (ISO string).
- `cursorId` (string, optional): The cursor for pagination based on the order ID.
- `startDate` (string, optional): The start date for filtering orders (ISO string).
- `endDate` (string, optional): The end date for filtering orders (ISO string).
- `status` (match_order_status, optional): The status of the orders to filter by.
- `storeId` (string, optional): The store ID to filter orders by.
- `matchId` (string, optional): The match ID to filter orders by.

### Returns

A promise that resolves to a Response object containing an array of match order items (see `match_order_item_response` type) or an error.

## getMatchOrdersSummary

Get a summary of match orders with counts per status.

### Parameters

- `startDate` (string, optional): The start date for filtering orders (ISO string).
- `endDate` (string, optional): The end date for filtering orders (ISO string).
- `status` (match_order_status, optional): The status of the orders to filter by.
- `storeId` (string, optional): The store ID to filter orders by.
- `matchId` (string, optional): The match ID to filter orders by.
- `searchQuery` (string, optional): The search query to match against customer names or order item names.

### Returns

A promise that resolves to a Response object containing a match order summary (see `match_order_summary_response` type) or an error.