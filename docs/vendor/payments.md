# Payment Management API

This document provides the complete API implementation and integration guide for payment-related operations for vendors.

## Overview

The payment system consists of two main components:

1. **Payout Accounts** - Bank accounts where vendors can withdraw their earnings. These are linked to stores for receiving payouts.
2. **Wallets** - Balance containers for users and stores. Users have personal wallets; stores have store wallets that accumulate revenue for vendor payouts.

## API Implementation

```typescript
import { supabase } from "@/lib/supabase";
import { Response } from "@/types/api";
import { Tables, TablesInsert, TablesUpdate } from '@/types/database.types';

/**
 * Get banks from Paystack API
 * Requires app client secret header
 * @param country - Country code (default: nigeria)
 * @param type - Bank type (default: nuban)
 * @returns List of banks
 */
const getBanks = async (
    country: string = "nigeria",
    type: string = "nuban"
): Promise<Response<any[]>> => {
    try {
        const supabaseUrl = process.env.EXPO_PUBLIC_SUPABASE_PROJECT_URL;
        if (!supabaseUrl) {
        throw new Error("Supabase URL not configured");
        }

        const url = `${supabaseUrl}/functions/v1/get-banks?country=${encodeURIComponent(country)}&type=${encodeURIComponent(type)}`;

        const response = await fetch(url, {
            method: "GET",
            headers: {
                "x-app-client-secret": process.env.EXPO_PUBLIC_APP_CLIENT_SECRET ?? "",
            },
        });

        if (!response.ok) {
            return {
                data: [],
                isSuccessful: false,
                message: `HTTP error! status: ${response.status}`,
            };
        }

        const data = await response.json();
        return { data, isSuccessful: true, message: "Banks fetched successfully" };
    } catch (error) {
        throw error;
    }
};

/**
 * Redeem booking code via edge function
 * Requires vendor authentication
 * @param code - Booking code to redeem
 * @returns Redemption result
 */
const redeemBookingCode = async (code: string): Promise<Response<any>> => {
    try {
        const { data, error } = await supabase.functions.invoke("redeem-booking-code", {
            body: { code },
        });

        if (error) {
            const errorBody = await error.context?.json();
            console.log("Detailed Function Error:", errorBody);
            throw error;
        }


        return { data, isSuccessful: true, message: "Booking code redeemed successfully" };
    } catch (error) {
        throw error;
    }
};

/**
 * Validate bank accounts via edge function
 * Requires app client secret header and user authentication (vendor only)
 * @param params - Bank account validation parameters
 * @returns Validation result
 */
const validateBankAccounts = async (
    params: {
        account_number: string;
        bank_code: string;
        bank_name?: string;
        save?: boolean;
    }
): Promise<Response<any>> => {
    try {
        const { data, error } = await supabase.functions.invoke("validate-bank-account", {
            body: params,
            headers: {
                "x-app-client-secret": process.env.EXPO_PUBLIC_APP_CLIENT_SECRET ?? "",
            },
        });

        if (error) {
            const errorBody = await error.context?.json();
            console.log("Detailed Function Error:", errorBody);
            throw error;
        }


        return { data, isSuccessful: true, message: "Bank account validated successfully" };
    } catch (error) {
        throw error;
    }
};

/**
 * Insert a new payout account
 * @param payoutAccountData - The payout account data to insert
 * @returns The inserted payout account or error
 */
const insertPayoutAccount = async (
    payoutAccountData: TablesInsert<'payout_accounts'>
): Promise<Response<Tables<'payout_accounts'>>> => {
    try {
        const { data, error } = await supabase
            .from('payout_accounts')
            .insert(payoutAccountData)
            .select()
            .single();

        if (error) throw error;

        return {
            isSuccessful: true,
            message: 'Payout account inserted successfully',
            data,
        };
    } catch (error) {
        console.error('Error inserting payout account:', error);
        throw error;
    }
};

/**
 * Update an existing payout account
 * @param payoutAccountId - The ID of the payout account to update
 * @param payoutAccountData - The payout account data to update
 * @returns The updated payout account or error
 */
const updatePayoutAccount = async (
    payoutAccountId: string,
    payoutAccountData: TablesUpdate<'payout_accounts'>
): Promise<Response<Tables<'payout_accounts'>>> => {
    try {
        const { data, error } = await supabase
            .from('payout_accounts')
            .update(payoutAccountData)
            .eq('id', payoutAccountId)
            .select()
            .single();

        if (error) throw error;

        return {
            isSuccessful: true,
            message: 'Payout account updated successfully',
            data,
        };
    } catch (error) {
        console.error('Error updating payout account:', error);
        throw error;
    }
};

/**
 * Soft delete a payout account by setting is_active to false
 * @param payoutAccountId - The ID of the payout account to deactivate
 * @returns The deactivated payout account or error
 */
const deactivatePayoutAccount = async (
    payoutAccountId: string
): Promise<Response<Tables<'payout_accounts'>>> => {
    try {
        const { data, error } = await supabase
            .from('payout_accounts')
            .update({ is_active: false, is_default: false })
            .eq('id', payoutAccountId)
            .select()
            .single();

        if (error) throw error;

        return {
            isSuccessful: true,
            message: 'Payout account deactivated successfully',
            data,
        };
    } catch (error) {
        console.error('Error deactivating payout account:', error);
        throw error;
    }
};

/**
 * Hard delete a payout account
 * @param payoutAccountId - The ID of the payout account to delete
 * @returns Success or error
 */
const deletePayoutAccount = async (
    payoutAccountId: string
): Promise<Response<void>> => {
    try {
        const { error } = await supabase
            .from('payout_accounts')
            .delete()
            .eq('id', payoutAccountId);

        if (error) throw error;

        return {
            isSuccessful: true,
            message: 'Payout account deleted successfully',
            data: undefined,
        };
    } catch (error) {
        console.error('Error deleting payout account:', error);
        throw error;
    }
};

/**
 * Get a single payout account by ID
 * @param payoutAccountId - The ID of the payout account to retrieve
 * @returns The payout account or error
 */
const getPayoutAccountById = async (
    payoutAccountId: string
): Promise<Response<Tables<'payout_accounts'>>> => {
    try {
        const { data, error } = await supabase
            .from('payout_accounts')
            .select()
            .eq('id', payoutAccountId)
            .single();

        if (error) throw error;

        return {
            isSuccessful: true,
            message: 'Payout account retrieved successfully',
            data,
        };
    } catch (error) {
        console.error('Error retrieving payout account:', error);
        throw error;
    }
};

/**
 * Get all active payout accounts for a store
 * @param storeId - The ID of the store to get payout accounts for
 * @returns The payout accounts or error
 */
const getPayoutAccountsByStoreId = async (
    storeId: string
): Promise<Response<Tables<'payout_accounts'>[]>> => {
    try {
        const { data, error } = await supabase
            .from('payout_accounts')
            .select()
            .eq('store_id', storeId)
            .eq('is_active', true)
            .order('is_default', { ascending: false })
            .order('created_at', { ascending: false });

        if (error) throw error;

        return {
            isSuccessful: true,
            message: 'Payout accounts retrieved successfully',
            data: data || [],
        };
    } catch (error) {
        console.error('Error retrieving payout accounts by store ID:', error);
        throw error;
    }
};

/**
 * Get the default payout account for a wallet
 * @param walletId - The ID of the wallet to get the default payout account for
 * @returns The default payout account or error
 */
const getDefaultPayoutAccountByWalletId = async (
    walletId: string
): Promise<Response<Tables<'payout_accounts'> | null>> => {
    try {
        const { data, error } = await supabase
            .from('payout_accounts')
            .select()
            .eq('wallet_id', walletId)
            .eq('is_default', true)
            .eq('is_active', true)
            .maybeSingle();

        if (error) throw error;

        return {
            isSuccessful: true,
            message: data
                ? 'Default payout account retrieved successfully'
                : 'No default payout account found',
            data,
        };
    } catch (error) {
        console.error('Error retrieving default payout account by wallet ID:', error);
        throw error;
    }
};

/**
 * Set a payout account as the default for its wallet
 * @param payoutAccountId - The ID of the payout account to set as default
 * @returns The updated payout account or error
 */
const setDefaultPayoutAccount = async (
  payoutAccountId: string
): Promise<Response<Tables<'payout_accounts'>>> => {
    try {
        const { data, error } = await supabase
            .from('payout_accounts')
            .update({ is_default: true })
            .eq('id', payoutAccountId)
            .select()
            .single();

        if (error) throw error;

        return {
            isSuccessful: true,
            message: 'Default payout account set successfully',
            data,
        };
    } catch (error) {
        console.error('Error setting default payout account:', error);
        throw error;
    }
};

/**
 * Get a wallet by profile ID (user wallet)
 * @param profileId - The ID of the profile to get the wallet for
 * @returns The wallet or error
 */
const getWalletByProfileId = async (
    profileId: string
): Promise<Response<Tables<'wallets'> | null>> => {
    try {
        const { data, error } = await supabase
            .from('wallets')
            .select()
            .eq('profile_id', profileId)
            .eq('owner_type', 'user')
            .eq('is_active', true)
            .maybeSingle();

        if (error) throw error;

        return {
        isSuccessful: true,
        message: data ? 'Wallet retrieved successfully' : 'No wallet found',
        data,
        };
    } catch (error) {
        console.error('Error retrieving wallet by profile ID:', error);
        throw error;
    }
};

/**
 * Get a wallet by store ID (store wallet)
 * @param storeId - The ID of the store to get the wallet for
 * @returns The wallet or error
 */
const getWalletByStoreId = async (
    storeId: string
): Promise<Response<Tables<'wallets'> | null>> => {
    try {
        const { data, error } = await supabase
        .from('wallets')
        .select()
        .eq('store_id', storeId)
        .eq('owner_type', 'store')
        .eq('is_active', true)
        .maybeSingle();

        if (error) throw error;

        return {
        isSuccessful: true,
        message: data ? 'Wallet retrieved successfully' : 'No wallet found',
        data,
        };
    } catch (error) {
        console.error('Error retrieving wallet by store ID:', error);
        throw error;
    }
};

export default {
    getBanks,
    redeemBookingCode,
    validateBankAccounts,
    insertPayoutAccount,
    updatePayoutAccount,
    deactivatePayoutAccount,
    deletePayoutAccount,
    getPayoutAccountById,
    getPayoutAccountsByStoreId,
    getDefaultPayoutAccountByWalletId,
    setDefaultPayoutAccount,
    getWalletByProfileId,
    getWalletByStoreId,
};
```

## Payout Account Management

### Link Bank Account for Withdrawals

Vendors link bank accounts to their stores for withdrawing earnings. When customers place orders, funds accumulate in the store's wallet. Vendors can then initiate withdrawals to their linked payout account.

### Step 1: Fetch Available Banks

First, retrieve the list of supported banks to populate a selection dropdown:

```typescript
import paymentApi from '@/api/handlePayments';

const banksResult = await paymentApi.getBanks("nigeria", "nuban");

if (banksResult.isSuccessful) {
    const banks = banksResult.data;
    // banks is an array of { code: string, name: string } objects
    console.log("Available banks:", banks.map(b => b.name));
} else {
    console.error("Failed to fetch banks:", banksResult.message);
}
```

### Step 2: Validate and Save Bank Account

Validate the vendor's bank account details before saving. This ensures the account number exists and matches the bank.

```typescript
import paymentApi from '@/api/handlePayments';

const validationResult = await paymentApi.validateBankAccounts({
    account_number: "1234567890",
    bank_code: "057", // Bank code from getBanks result
    bank_name: "Zenith Bank", // Optional, auto-filled from bank_code
    save: true // Set to true to automatically save validated account
});

if (validationResult.isSuccessful) {
    console.log("Account validated:", validationResult.data);
    // If save was true, the account is now saved to the database
} else {
    console.error("Validation failed:", validationResult.message);
}
```

### Step 3: Manually Insert Payout Account

If you prefer to validate and save in separate steps:

```typescript
import paymentApi from '@/api/handlePayments';

// First validate the account
const validationResult = await paymentApi.validateBankAccounts({
    account_number: "1234567890",
    bank_code: "057",
    save: false
});

if (validationResult.isSuccessful && validationResult.data?.account_name) {
    // Then manually insert
    const insertResult = await paymentApi.insertPayoutAccount({
        store_id: storeId,
        account_number: "1234567890",
        account_name: validationResult.data.account_name,
        bank_code: "057",
        bank_name: "Zenith Bank",
        is_active: true,
        is_default: true,
    });

    if (insertResult.isSuccessful) {
        console.log("Payout account saved:", insertResult.data);
    }
}
```

### Get Payout Accounts for a Store

Retrieve all active payout accounts associated with a store:

```typescript
import paymentApi from '@/api/handlePayments';

const result = await paymentApi.getPayoutAccountsByStoreId(storeId);

if (result.isSuccessful) {
    const accounts = result.data;
    // accounts are ordered by is_default (desc) and created_at (desc)
    console.log("Payout accounts:", accounts);
}
```

### Get Default Payout Account by Wallet

Retrieve the default payout account for a specific wallet:

```typescript
import paymentApi from '@/api/handlePayments';

// First get the store wallet
const walletResult = await paymentApi.getWalletByStoreId(storeId);
if (walletResult.isSuccessful && walletResult.data) {
    const walletId = walletResult.data.id;
    
    const result = await paymentApi.getDefaultPayoutAccountByWalletId(walletId);

    if (result.isSuccessful && result.data) {
        console.log("Default account:", result.data.account_name);
        console.log("Bank:", result.data.bank_name);
        console.log("Account number:", result.data.account_number);
    }
}
```

### Update Payout Account

Update an existing payout account:

```typescript
import paymentApi from '@/api/handlePayments';

const updateResult = await paymentApi.updatePayoutAccount(payoutAccountId, {
    account_name: "Updated Account Name",
});

if (updateResult.isSuccessful) {
    console.log("Account updated:", updateResult.data);
}
```

### Deactivate Payout Account

Soft delete a payout account (sets `is_active` to false and `is_default` to false):

```typescript
import paymentApi from '@/api/handlePayments';

const result = await paymentApi.deactivatePayoutAccount(payoutAccountId);

if (result.isSuccessful) {
    console.log("Account deactivated");
}
```

### Delete Payout Account

Hard delete a payout account from the database:

```typescript
import paymentApi from '@/api/handlePayments';

const result = await paymentApi.deletePayoutAccount(payoutAccountId);

if (result.isSuccessful) {
    console.log("Account permanently deleted");
}
```

### Set Default Payout Account

Set an existing payout account as the default for receiving withdrawals:

```typescript
import paymentApi from '@/api/handlePayments';

const result = await paymentApi.setDefaultPayoutAccount(payoutAccountId);

if (result.isSuccessful) {
    console.log("Default account updated:", result.data.account_name);
}
```

## Wallet Management

### Get Store Wallet

Retrieve the wallet associated with a store to check available balance for withdrawal:

```typescript
import paymentApi from '@/api/handlePayments';

const result = await paymentApi.getWalletByStoreId(storeId);

if (result.isSuccessful && result.data) {
    console.log("Store balance:", result.data.balance);
    console.log("Currency:", result.data.currency);
    console.log("Wallet ID:", result.data.id);
}
```

### Get User Wallet

Retrieve a user's personal wallet:

```typescript
import paymentApi from '@/api/handlePayments';

const result = await paymentApi.getWalletByProfileId(profileId);

if (result.isSuccessful && result.data) {
    console.log("User balance:", result.data.balance);
}
```

### Check Store Wallet Balance Before Withdrawal

```typescript
import paymentApi from '@/api/handlePayments';

async function canWithdraw(storeId: string, amount: number) {
    const walletResult = await paymentApi.getWalletByStoreId(storeId);
    
    if (!walletResult.isSuccessful || !walletResult.data) {
        throw new Error("Store wallet not found");
    }
    
    const availableBalance = Number(walletResult.data.balance);
    return availableBalance >= amount;
}
```

## Booking Code Redemption

Vendors can redeem booking codes to activate their store:

```typescript
import paymentApi from '@/api/handlePayments';

const result = await paymentApi.redeemBookingCode("BOOKING-CODE-123");

if (result.isSuccessful) {
    console.log("Booking redeemed successfully");
    console.log("Result:", result.data);
} else {
    console.error("Failed to redeem booking code:", result.message);
}
```

## Security: Shared Secret Verification

The `getBanks` and `validateBankAccounts` functions require a shared secret to ensure requests originate from the legitimate app client.

**Client-side (App):**
- Environment variable: `EXPO_PUBLIC_APP_CLIENT_SECRET`
- Sent in headers: `x-app-client-secret`

**Setup:**
1. Add `EXPO_PUBLIC_APP_CLIENT_SECRET` to your `.env` file
2. Ensure it matches the `APP_CLIENT_SECRET` configured in Supabase Edge Functions

## Payout Account Data Structure

### Tables<'payout_accounts'>

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | string (UUID) | Auto | Primary key |
| store_id | string (UUID) | Yes | The store this payout account belongs to |
| wallet_id | string (UUID) | Cond'l | The wallet this account is linked to for withdrawals |
| account_number | string | Yes | Bank account number |
| account_name | string | Yes | Verified account name |
| bank_code | string | Yes | Bank code (e.g., "057" for Zenith Bank) |
| bank_name | string | Yes | Bank name |
| is_active | boolean | Yes | Whether account is active |
| is_default | boolean | No | Whether this is the default payout account |
| created_at | string | Auto | Creation timestamp |

## Wallet Data Structure

### Tables<'wallets'>

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | string (UUID) | Auto | Primary key |
| store_id | string (UUID) | Cond'l | The store this wallet belongs to (for store wallets) |
| profile_id | string (UUID) | Cond'l | The user profile this wallet belongs to (for user wallets) |
| owner_type | string | Yes | Either "user" or "store" |
| balance | number | Yes | Current wallet balance |
| currency | string | Yes | Currency code (e.g., "NGN") |
| is_active | boolean | Yes | Whether wallet is active |
| created_at | string | Auto | Creation timestamp |

## Integration with Store Setup Flow

Integrate payout accounts into the store setup flow:

```typescript
import paymentApi from '@/api/handlePayments';

async function setupStorePayout(storeId: string, bankDetails: {
    account_number: string;
    bank_code: string;
}) {
    // Validate and save in one step
    const result = await paymentApi.validateBankAccounts({
        account_number: bankDetails.account_number,
        bank_code: bankDetails.bank_code,
        save: true
    });

    if (result.isSuccessful) {
        console.log("Store payout account configured successfully");
        return result.data;
    }

    throw new Error("Failed to setup payout account");
}
```

## Related Documentation

- [Store Management](/docs/vendor/store-management) - Store identity and operations
- [Menu Management](/docs/vendor/menu) - Menu and options management