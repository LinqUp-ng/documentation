# Phase 0: System Core (The Foundation)
*Establishing global configuration, security, and authentication.*

## 🏗️ Supabase Infrastructure
This guide covers the technical initialization of the Supabase client and the core infrastructure configurations for LinqUp.

### 1. Client Initialization
We enforce strict typing across the application by using the generated `Database` types.

```typescript
import { createClient } from "@supabase/supabase-js";
import { Database } from "@/types/database.types.ts";

const supabaseUrl = process.env.EXPO_PUBLIC_SUPABASE_URL!;
const supabaseAnonKey = process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY!;

export const supabase = createClient<Database>(supabaseUrl, supabaseAnonKey);
```

### 2. API Response Structure
All API handlers must follow this standardized response format for consistency across the platform.

```typescript
export type Response<T> = {
    isSuccessful: boolean;
    message: string;
    data: T;
};
```

### 3. Persistent Storage

The storage engine for auth tokens differs based on the target platform:

#### 📱 Mobile (React Native / Expo)
To maintain user sessions between app restarts, you must use `AsyncStorage`.
```bash
npx expo install @react-native-async-storage/async-storage
```

#### 🌐 Web (Next.js / React)
For web deployments, use **standard browser storage** or a **cookie-based approach**.
> [!WARNING]
> Do NOT use `@react-native-async-storage/async-storage` on the web; it is only compatible with native mobile environments.

---

## 🚀 Roadmap Phase 0
### 0. Infrastructure
- [x] **Supabase Setup:** Database setup, auth config, and storage buckets.

### 1. Global Platform Settings
- [x] **Table:** `public.global_settings` (earning %, min withdrawal, maintenance mode).
- [ ] **Endpoint:** `GET /admin/settings` - Retrieve all configurations.
- [ ] **Endpoint:** `GET /admin/settings/:key` - Fetch specific setting by key.
- [ ] **Endpoint:** `GET /admin/settings/id/:id` - Fetch specific setting by UUID.
- [ ] **Endpoint:** `PUT /admin/settings/:key` - Update setting by key.
- [ ] **Endpoint:** `PUT /admin/settings/id/:id` - Update setting by UUID.
- [ ] **Endpoint:** `POST /admin/settings` - Create a new system setting.

#### Implementation: `api/handleGlobalSettings.ts`
```typescript
import { supabase, Tables } from "@/lib/supabase";
import { Response } from "@/types/api";

export type SettingDataType = "string" | "number" | "boolean";
export type GlobalSetting = Tables<"global_settings">;

// 1. Get All Settings
const getAllSettings = async (): Promise<Response<GlobalSetting[]>> => {
    const { data, error } = await supabase.from("global_settings").select("*");
    if (error) throw error;
    return { isSuccessful: true, message: "Settings fetched", data: data || [] };
};

// 2. Get by Key
const getSettingByKey = async (key: string): Promise<Response<GlobalSetting>> => {
    const { data, error } = await supabase.from("global_settings").select("*").eq("key", key).single();
    if (error) throw error;
    return { isSuccessful: true, message: `Fetched ${key}`, data };
};

// 3. Update by Key
const updateSettingByKey = async (key: string, payload: Partial<GlobalSetting>) => {
    const { data, error } = await supabase.from("global_settings").update(payload).eq("key", key).select("*").single();
    if (error) throw error;
    return { isSuccessful: true, message: `Updated ${key}`, data };
};

// 4. Get by ID
const getSettingById = async (id: string): Promise<Response<GlobalSetting>> => {
    const { data, error } = await supabase.from("global_settings").select("*").eq("id", id).single();
    if (error) throw error;
    return { isSuccessful: true, message: "Fetched by ID", data };
};

// 5. Update by ID
const updateSettingById = async (id: string, payload: Partial<GlobalSetting>) => {
    const { data, error } = await supabase.from("global_settings").update(payload).eq("id", id).select("*").single();
    if (error) throw error;
    return { isSuccessful: true, message: "Updated by ID", data };
};

// 6. Create New
const createNewSetting = async (payload: Omit<GlobalSetting, "id" | "created_at" | "updated_at">) => {
    const { data, error } = await supabase.from("global_settings").insert(payload).select("*").single();
    if (error) throw error;
    return { isSuccessful: true, message: "Created", data };
};
```

### 2. Authentication & Identity Utilities
The platform uses a unified authentication handler to manage sessions, password changes, and OTP flows.

#### Implementation: `api/handleAuth.ts`
```typescript
import { supabase } from "../lib/supabase";

// 1. Session Management
const login = async ({ email, password }) => {
    const { data, error } = await supabase.auth.signInWithPassword({ email, password });
    if (error) throw error;
    return { isSuccessful: true, message: "login successful", data };
};

const signout = async () => {
    const { error } = await supabase.auth.signOut();
    if (error) throw error;
    return { isSuccessful: true, message: "Signed out successfully", data: null };
};

// 2. Auth OTP (Supabase Native)
const verifyAuthOTP = async ({ email, otp, type }) => {
    const { data, error } = await supabase.auth.verifyOtp({ email, token: otp, type });
    if (error) throw error;
    return { isSuccessful: true, message: "Verification successful", data };
};

const resendAuthVerificationOTP = async ({ email, type = "signup" }) => {
    const { data, error } = await supabase.auth.resend({ email, type });
    if (error) throw error;
    return { isSuccessful: true, message: "Email resent", data };
};

// 3. Custom OTP Flow (Edge Functions)
// Note: Generated OTP codes are valid for 10 minutes.
const sendOTP = async (payload: { email?: string; user_id?: string }) => {
    const { data, error } = await supabase.functions.invoke("send-otp", {
        body: payload,
        headers: { "x-client-secret": process.env.EXPO_PUBLIC_APP_CLIENT_SECRET ?? "" }
    });
    if (error) throw error;
    return { isSuccessful: true, message: "OTP sent successfully", data };
};

const verifyCustomOTP = async (payload: { email?: string; user_id?: string; otp: string }) => {
    const { data, error } = await supabase.functions.invoke("verify-otp", {
        body: payload,
        headers: { "x-client-secret": process.env.EXPO_PUBLIC_APP_CLIENT_SECRET ?? "" }
    });
    if (error) throw error;
    return { isSuccessful: true, message: "OTP verified successfully", data };
};

// 4. Security & Recovery
const changePassword = async ({ current_password, new_password, email }) => {
    // Re-authenticate first to verify current password
    const { error: loginError } = await supabase.auth.signInWithPassword({ email, password: current_password });
    if (loginError) throw new Error("Invalid current password");

    const { data, error } = await supabase.auth.updateUser({ password: new_password });
    if (error) throw error;
    return { isSuccessful: true, message: "Password updated successfully", data: data?.user };
};

const requestPasswordReset = async (email: string) => {
    const { error } = await supabase.auth.resetPasswordForEmail(email);
    if (error) throw error;
    return { isSuccessful: true, message: "Reset instructions sent", data: null };
};
```

---

## 📧 Email & Communication
### 1. Email Templates System
- **Table:** `public.email_templates` - Stores HTML content with dynamic placeholders (e.g., `{{ .Token }}`).
- **Template Types:** `otp`, `password`, `refund`, `date_reminder`, `order_created`.

### 2. Resend Integration
- **Service:** [Resend](https://resend.com)
- **Primary Function:** `send-otp` - Fetches the template, replaces keywords, and dispatches via Resend API.
- **Config:** Requires `RESEND_API_KEY` set in Supabase Edge Function secrets.

---

