# Vendor Onboarding

This document covers the complete vendor onboarding flow from initial setup to account activation.

## Overview

The vendor onboarding process involves three main steps:

1. **Admin creates business** - An administrator adds a new business using `addBusiness`, which generates an onboarding token
2. **Vendor retrieves onboarding details** - The vendor uses the token to get their account information via `getOnboardingVendor`
3. **Vendor completes signup** - The vendor sets their password to activate their account via `completeVendorSignup`
4. **Vendor logs in** - After signup, the vendor must log in using the standard authentication endpoint

## Step 1: Get Onboarding Details

Retrieves the vendor's onboarding information using the token provided by the administrator. This is a public endpoint that does not require authentication but requires the app client secret.

### Function: `getOnboardingVendor`

```typescript
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
```

### Request

```typescript
{
    token: string;  // The onboarding token provided by the admin
}
```

### Response (OnboardingVendorData)

```typescript
{
    email: string;
    business_name: string;
    business_status: string;
    first_name: string | null;
    last_name: string | null;
    vendor_role: string;  // "owner" | "manager" | "member"
    created_at: string;
}
```

### Usage Example

```typescript
import { handleVendors } from '@/api/handleVendors';

const result = await handleVendors.getOnboardingVendor(token);

if (result.isSuccessful) {
    console.log("Business:", result.data.business_name);
    console.log("Email:", result.data.email);
    console.log("Role:", result.data.vendor_role);
    console.log("Name:", result.data.first_name, result.data.last_name);
} else {
    console.error("Failed to fetch onboarding data:", result.message);
}
```

---

## Step 2: Complete Vendor Signup

Completes the vendor signup by setting a password and activating the account. After this step, the vendor account becomes active.

### Function: `completeVendorSignup`

```typescript
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

        if (error) throw error;

        return data as CompleteVendorSignupResponse;
    } catch (error) {
        throw error;
    }
};
```

### Request Body (CompleteVendorSignupParams)

```typescript
{
    token: string;      // The onboarding token
    password: string;   // The password the vendor wants to set
}
```

### Response (CompleteVendorSignupResponse)

```typescript
{
    success: boolean;
    message: string;
    userId: string;
    session?: any;
    user?: any;
}
```

### Usage Example

```typescript
import { handleVendors } from '@/api/handleVendors';

const result = await handleVendors.completeVendorSignup({
    token: onboardingToken,
    password: "securePassword123"
});

if (result.success) {
    console.log("Account activated for user:", result.userId);
    // Proceed to login
} else {
    console.error("Failed to complete signup:", result.message);
}
```

---

## Step 3: Login

**IMPORTANT:** After completing the vendor signup, you must call the traditional login endpoint to authenticate and start your session. The `completeVendorSignup` function does not automatically log you in.

### Login Using Supabase Auth

```typescript
import { supabase } from '@/lib/supabase';

const { data, error } = await supabase.auth.signInWithPassword({
    email: vendorEmail,
    password: vendorPassword,
});

if (error) {
    console.error("Login failed:", error.message);
} else {
    console.log("Logged in successfully:", data.user);
    console.log("Session:", data.session);
}
```

---

## Complete Onboarding Flow

Here's the complete flow from start to finish:

```typescript
import { handleVendors } from '@/api/handleVendors';
import { supabase } from '@/lib/supabase';

async function completeVendorOnboarding(onboardingToken: string, password: string) {
    // Step 1: Get onboarding details
    const onboardingResult = await handleVendors.getOnboardingVendor(onboardingToken);
    if (!onboardingResult.isSuccessful) {
        console.error("Failed to get onboarding details:", onboardingResult.message);
        throw new Error("Invalid or expired onboarding token");
    }

    const { email, business_name, vendor_role, first_name, last_name } = onboardingResult.data;
    console.log("Onboarding details fetched for:", first_name, last_name);
    console.log("Business:", business_name);
    console.log("Role:", vendor_role);

    // Step 2: Complete signup with password
    const signupResult = await handleVendors.completeVendorSignup({
        token: onboardingToken,
        password: password
    });

    if (!signupResult.success) {
        console.error("Failed to complete signup:", signupResult.message);
        throw new Error("Failed to complete signup");
    }

    console.log("Signup completed. User ID:", signupResult.userId);

    // Step 3: Login with traditional auth endpoint
    const { data, error } = await supabase.auth.signInWithPassword({
        email: email,
        password: password
    });

    if (error) {
        console.error("Login failed:", error.message);
        throw new Error("Login failed");
    }

    console.log("Vendor successfully onboarded and logged in!");
    console.log("User:", data.user);
    console.log("Session:", data.session);

    return {
        user: data.user,
        session: data.session,
        business: business_name,
        role: vendor_role
    };
}

// Usage
try {
    const result = await completeVendorOnboarding("abc123token", "securePassword123");
    // Proceed with authenticated session
} catch (error) {
    console.error("Onboarding failed:", error);
}
```

---

## Error Handling

Always handle errors at each step of the onboarding process:

```typescript
import { handleVendors } from '@/api/handleVendors';
import { supabase } from '@/lib/supabase';

// Step 1: Error handling for getOnboardingVendor
const onboardingResult = await handleVendors.getOnboardingVendor(token);
if (!onboardingResult.isSuccessful) {
    // Token may be invalid or expired
    alert("Invalid or expired onboarding token. Please contact your administrator.");
    return;
}

// Step 2: Error handling for completeVendorSignup
try {
    const signupResult = await handleVendors.completeVendorSignup({
        token: token,
        password: password
    });
    
    if (!signupResult.success) {
        alert("Failed to complete signup: " + signupResult.message);
        return;
    }
} catch (error) {
    // Handle network errors or validation errors
    alert("Failed to complete signup. Please check your connection and try again.");
    return;
}

// Step 3: Error handling for login
const { error } = await supabase.auth.signInWithPassword({
    email: email,
    password: password
});

if (error) {
    alert("Login failed. Please try again.");
    return;
}
```

### Common Error Scenarios

| Scenario | Error Message | Solution |
|----------|---------------|----------|
| Invalid/expired token | "Invalid or expired token" | Request a new invitation from admin |
| Token already used | "Token has already been used" | Contact admin to resend invitation |
| Weak password | "Password does not meet requirements" | Use a stronger password |
| Network error | "Failed to fetch onboarding data" | Check internet connection |
| App client secret mismatch | "Unauthorized" | Ensure EXPO_PUBLIC_APP_CLIENT_SECRET is configured correctly |

---

## Security: Shared Secret Verification

The public onboarding functions (`get-onboarding-vendor`, `complete-vendor-signup`) require a shared secret to ensure requests originate from the legitimate app client.

**Server-side (Supabase Edge Functions):**
- Environment variable: `APP_CLIENT_SECRET`
- Header to check: `x-app-client-secret`
- Returns 401 Unauthorized if missing or mismatched

**Client-side (App):**
- Environment variable: `EXPO_PUBLIC_APP_CLIENT_SECRET`
- Sent in headers: `x-app-client-secret`

**Setup:**
1. Set `APP_CLIENT_SECRET` as a secret in your Supabase project
2. Add `EXPO_PUBLIC_APP_CLIENT_SECRET` to your `.env` file with the same value

---

## Password Requirements

Passwords must meet the following security requirements:

- Minimum 8 characters
- At least one uppercase letter
- At least one lowercase letter
- At least one number
- At least one special character

---

## Notes

- The onboarding token is single-use and expires after the signup is completed
- After signup completion, the vendor must immediately login using the traditional auth endpoint
- The vendor's role determines their permissions within the business:
  - **owner**: Full access to the business and all stores
  - **manager**: Operational access to manage products, orders, and staff
  - **member**: Limited access based on specific permissions
- If a vendor needs to reset their password after onboarding, they should use the standard password reset flow
- The onboarding token should be securely shared with the vendor (e.g., via email with a secure link)

---

## API Reference

For complete API documentation, see:
- [Vendor Management](/docs/admin/vendors) - Admin functions for managing vendors and businesses
- [Admin Onboarding](/docs/admin/onboarding) - Admin account management and roles