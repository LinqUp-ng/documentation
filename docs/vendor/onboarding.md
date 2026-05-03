# Vendor Onboarding

This document covers the complete vendor onboarding flow from initial setup to account activation.

## Overview

The vendor onboarding process involves three main steps:

1. **Admin creates business** - An administrator adds a new business, which generates an onboarding token
2. **Vendor retrieves onboarding details** - The vendor uses the token to get their account information
3. **Vendor completes signup** - The vendor sets their password to activate their account

## Step 1: Get Onboarding Details

Retrieves the vendor's onboarding information using the token provided by the administrator. This is a public endpoint that does not require authentication.

### Endpoint
`POST /functions/v1/get-onboarding-vendor`

### Request Body

```typescript
{
  token: string;
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
  vendor_role: string;
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
}
```

## Step 2: Complete Vendor Signup

Completes the vendor signup by setting a password and activating the account. After this step, the vendor account becomes active.

### Endpoint
`POST /functions/v1/complete-vendor-signup`

### Request Body (CompleteVendorSignupParams)

```typescript
{
  token: string;
  password: string;
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
}
```

## Step 3: Login

**IMPORTANT:** After completing the vendor signup, you must call the traditional login endpoint to authenticate and start your session.

### Login Endpoint

Use the standard authentication login endpoint with the vendor's email and the password they just set.

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
}
```

## Complete Onboarding Flow

Here's the complete flow from start to finish:

```typescript
import { handleVendors } from '@/api/handleVendors';
import { supabase } from '@/lib/supabase';

// Step 1: Get onboarding details
const onboardingResult = await handleVendors.getOnboardingVendor(token);
if (!onboardingResult.isSuccessful) {
  console.error("Failed to get onboarding details:", onboardingResult.message);
  return;
}

const { email, business_name, vendor_role } = onboardingResult.data;

// Step 2: Complete signup with password
const signupResult = await handleVendors.completeVendorSignup({
  token: token,
  password: "securePassword123"
});

if (!signupResult.success) {
  console.error("Failed to complete signup:", signupResult.message);
  return;
}

// Step 3: Login with traditional auth endpoint
const { data, error } = await supabase.auth.signInWithPassword({
  email: email,
  password: "securePassword123"
});

if (error) {
  console.error("Login failed:", error.message);
} else {
  console.log("Vendor successfully onboarded and logged in!");
}
```

## Error Handling

Always handle errors at each step of the onboarding process:

```typescript
// Step 1
const onboardingResult = await handleVendors.getOnboardingVendor(token);
if (!onboardingResult.isSuccessful) {
  // Token may be invalid or expired
  alert("Invalid or expired onboarding token");
  return;
}

// Step 2
try {
  const signupResult = await handleVendors.completeVendorSignup({
    token: token,
    password: password
  });
} catch (error) {
  // Handle network errors or validation errors
  alert("Failed to complete signup");
  return;
}

// Step 3
const { error } = await supabase.auth.signInWithPassword({
  email: email,
  password: password
});
if (error) {
  alert("Login failed. Please try again.");
  return;
}
```

## Notes

- The onboarding token is single-use and expires after the signup is completed
- Password must meet the system's security requirements
- After signup completion, the vendor should immediately login using the traditional auth endpoint
- The vendor's role determines their permissions within the business (owner, manager, member)
