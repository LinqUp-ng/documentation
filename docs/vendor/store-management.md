# Phase 2: Store Content (The "Content" Layer)
*Vendors must prepare their stores and menus before user interaction.*

### 1. Store Identity
- [ ] **Create:** `POST /vendor/stores` - Create a new store. **Only business owners can create stores**. The authenticated user is automatically added as a default vendor in the created store.

```typescript
import { supabase, Tables } from "@/lib/supabase";
import { Response } from "@/types/api";
import { TablesInsert } from "@/types/database.types";

const create = async (payload: TablesInsert<'stores'>): Promise<Response<Tables<'stores'>>> => {
    try {
        const { data, error } = await supabase.from('stores')
        .insert(payload)
        .select("*")
        .single();

        if (error) {
            throw error;
        }

        return {
            data,
            isSuccessful: true,
            message: "Store create successfully"
        }
    } catch (error) {
        throw error
    }
}
```
- [ ] **Identity:** `PUT /vendor/stores/:id` - Update logo, cover photo, and address.
- [ ] **Operations:** `GET /vendor/stores/:id/hours`, `PUT /vendor/stores/:id/hours` (Working days).
- [ ] **Finance:** `POST /vendor/finance/payout-account` - Link Paystack-compatible account. See [Payment Management](/docs/vendor/payments) for details.

### 2. Menu Management
- [ ] **Categorization:** `public.food_categories` - Starters, Mains, Drinks.
- [ ] **Catalog:** `POST /vendor/menu`, `PUT /vendor/menu/:id` - Item with price, stock, and descriptions.
- [ ] **Options/Customization:** `POST /vendor/menu/options` - Required sides/extras.
