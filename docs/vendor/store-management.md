# Phase 2: Store Content (The "Content" Layer)
*Vendors must prepare their stores and menus before user interaction.*

### 1. Store Identity
- [ ] **Identity:** `PUT /vendor/stores/:id` - Update logo, cover photo, and address.
- [ ] **Operations:** `GET /vendor/stores/:id/hours`, `PUT /vendor/stores/:id/hours` (Working days).
- [ ] **Finance:** `POST /vendor/finance/payout-account` - Link Paystack-compatible account.

### 2. Menu Management
- [ ] **Categorization:** `public.food_categories` - Starters, Mains, Drinks.
- [ ] **Catalog:** `POST /vendor/menu`, `PUT /vendor/menu/:id` - Item with price, stock, and descriptions.
- [ ] **Options/Customization:** `POST /vendor/menu/options` - Required sides/extras.
