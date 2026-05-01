# Phase 0: System Core (The Foundation)

_Establishing global configuration, security, and authentication._

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

// 3. (Optional but Recommended) Export table-specific types for convenience
export type Tables<T extends keyof Database["public"]["Tables"]> =
    Database["public"]["Tables"][T]["Row"];
export type Enums<T extends keyof Database["public"]["Enums"]> =
    Database["public"]["Enums"][T];

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

> [!IMPORTANT]
> **Serverless Architecture:** LinqUp uses a **Serverless / SDK-First** approach. While endpoints are listed using REST conventions (e.g., `GET /resource`) for clarity, developers should **NOT** use Axios or Fetch. Always use the provided **Supabase SDK** utilities documented in each section to ensure proper type safety and RLS authentication.

## 🚀 Roadmap Phase 0

### 0. Infrastructure

- [x] **Supabase Setup:** Database setup, auth config, and storage buckets.

### 1. Global Platform Settings

- [x] **Table:** `public.global_settings` (earning %, min withdrawal, maintenance mode).
- [x] **Endpoint:** `GET /admin/settings` - Retrieve all configurations.
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
const getSettingByKey = async (
  key: string,
): Promise<Response<GlobalSetting>> => {
  const { data, error } = await supabase
    .from("global_settings")
    .select("*")
    .eq("key", key)
    .single();
  if (error) throw error;
  return { isSuccessful: true, message: `Fetched ${key}`, data };
};

// 3. Update by Key
const updateSettingByKey = async (
  key: string,
  payload: Partial<GlobalSetting>,
) => {
  const { data, error } = await supabase
    .from("global_settings")
    .update(payload)
    .eq("key", key)
    .select("*")
    .single();
  if (error) throw error;
  return { isSuccessful: true, message: `Updated ${key}`, data };
};

// 4. Get by ID
const getSettingById = async (id: string): Promise<Response<GlobalSetting>> => {
  const { data, error } = await supabase
    .from("global_settings")
    .select("*")
    .eq("id", id)
    .single();
  if (error) throw error;
  return { isSuccessful: true, message: "Fetched by ID", data };
};

// 5. Update by ID
const updateSettingById = async (
  id: string,
  payload: Partial<GlobalSetting>,
) => {
  const { data, error } = await supabase
    .from("global_settings")
    .update(payload)
    .eq("id", id)
    .select("*")
    .single();
  if (error) throw error;
  return { isSuccessful: true, message: "Updated by ID", data };
};

// 6. Create New
const createNewSetting = async (
  payload: Omit<GlobalSetting, "id" | "created_at" | "updated_at">,
) => {
  const { data, error } = await supabase
    .from("global_settings")
    .insert(payload)
    .select("*")
    .single();
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
  const { data, error } = await supabase.auth.signInWithPassword({
    email,
    password,
  });
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
  const { data, error } = await supabase.auth.verifyOtp({
    email,
    token: otp,
    type,
  });
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
    headers: {
      "x-client-secret": process.env.EXPO_PUBLIC_APP_CLIENT_SECRET ?? "",
    },
  });
  if (error) throw error;
  return { isSuccessful: true, message: "OTP sent successfully", data };
};

const verifyCustomOTP = async (payload: {
  email?: string;
  user_id?: string;
  otp: string;
}) => {
  const { data, error } = await supabase.functions.invoke("verify-otp", {
    body: payload,
    headers: {
      "x-client-secret": process.env.EXPO_PUBLIC_APP_CLIENT_SECRET ?? "",
    },
  });
  if (error) throw error;
  return { isSuccessful: true, message: "OTP verified successfully", data };
};

// 4. Security & Recovery
const changePassword = async ({ current_password, new_password, email }) => {
  // Re-authenticate first to verify current password
  const { error: loginError } = await supabase.auth.signInWithPassword({
    email,
    password: current_password,
  });
  if (loginError) throw new Error("Invalid current password");

  const { data, error } = await supabase.auth.updateUser({
    password: new_password,
  });
  if (error) throw error;
  return {
    isSuccessful: true,
    message: "Password updated successfully",
    data: data?.user,
  };
};

const requestPasswordReset = async (email: string) => {
  const { error } = await supabase.auth.resetPasswordForEmail(email);
  if (error) throw error;
  return { isSuccessful: true, message: "Reset instructions sent", data: null };
};
```

---

## 🛠️ Utilities & File Management

LinqUp uses a centralized storage and image processing system to ensure fast loading and consistent media handling across platforms.

### 1. Storage Buckets & Privacy

Entity images are stored in dedicated buckets based on their type.

| Bucket       | Entity                     | Access Control                 |
| :----------- | :------------------------- | :----------------------------- |
| `profiles`   | User Avatars               | **Public**                     |
| `stores`     | Store images/banners       | **Public**                     |
| `admins`     | Admin profile media        | **Public**                     |
| `vendors`    | Vendor documentation/media | **Public**                     |
| `businesses` | Business logos/assets      | **Public**                     |
| `chats`      | Message attachments        | **Private** (Signed URLs only) |
| `apps`       | Platform assets            | **Public**                     |

### 2. Storage Security (RLS)

Access to storage folders is strictly controlled via PostgreSQL RLS policies that leverage custom JWT claims.

- **`stores`**: Restricted to **Admins** and **Vendors** belonging to the business that owns the store.
- **`admins`, `vendors`**: Users can only manage files within a folder named after their `auth.uid`.
- **`businesses`**: Restricted to **Admins** and team members belonging to that specific `business_id`.
- **`chats`**: Private; users can only manage files within their own `auth.uid` folder.

### 3. One-Shot Image Processing

To provide a premium experience and optimize storage, LinqUp uses a unified "One-Shot" Edge Function for all image uploads.

#### Workflow:

1.  **Input:** Client sends raw image data and an **Entity ID** to `upload-with-blurhash`.
    - _Note: The `id` should be the UUID of the entity (user, store, etc.). If creating a new entity, generate a local UUID using the `uuidv4` library before uploading._
2.  **WebP Conversion:** The server automatically converts the image to **WebP (85% quality)**.
3.  **Intelligent Resizing:** If the image width exceeds **1024px**, it is scaled down to 1024px while maintaining aspect ratio.
4.  **Blurhash Generation:** The function creates a 32x32 thumbnail in-memory to generate a Blurhash sequence.
5.  **Direct Upload:** The finalized WebP is uploaded to the designated bucket at `[entity_id]/[filename]-[timestamp].webp`.
6.  **Response:** The client receives:
    - `uri`: The storage path (e.g., `user_123/banner-17145700.webp`).
    - `blur_hash`: The hash string for the `BlurView`.
    - `width`/`height`: The original dimensions of the image.

#### Cross-Platform Strategy:

- **Mobile & Web:** Both platforms should use the same `api/handleUpload.ts` utilities. This ensures that every image in the system follows the exact same compression and placeholder standards without needing specific libraries in the Web or Mobile app repos.

### 3. Implementation: `api/handleUpload.ts`

Full implementation for file management, signature generation, and cloud-based image processing.

```typescript
import { supabase } from "../lib/supabase";

// 1. Simple Single File Upload
const uploadFile = async ({ id, uri, mimeType, bucketName }) => {
  const arraybuffer = await fetch(uri).then((res) => res.arrayBuffer());
  const fileExt = uri.split(".").pop();
  const path = `${id}/${Date.now()}.${fileExt}`;

  const { data, error } = await supabase.storage
    .from(bucketName)
    .upload(path, arraybuffer, { contentType: mimeType });

  if (error) throw error;
  return { uri: data.path, isSuccessful: true };
};

// 2. Upload with Automatic Server-Side Blurhash (Recommended)
// This calls a Deno Edge function to generate blurhash and upload in one step
const uploadWithBlurhash = async ({ id, uri, bucketName, fileName }) => {
  const arraybuffer = await fetch(uri).then((res) => res.arrayBuffer());
  const { data, error } = await supabase.functions.invoke(
    "upload-with-blurhash",
    {
      body: arraybuffer,
      headers: {
        "x-bucket-name": bucketName,
        "x-user-id": id,
        "x-file-name": fileName || "image",
        "content-type": "image/webp",
      },
    },
  );
  if (error) throw error;
  return data; // Returns { uri, blur_hash, width, height }
};

// 3. Bulk Blurhash Generation (For existing images or Next.js server-side)
const generateBlurhashesBulk = async (base64Images: string[]) => {
  const { data, error } = await supabase.functions.invoke(
    "generate-blurhash-bulk",
    {
      body: { images: base64Images },
    },
  );
  if (error) throw error;
  return data; // Array of blurhash strings
};

// 4. File Deletion
const deleteFile = async ({ bucketName, uri }) => {
  const { error } = await supabase.storage.from(bucketName).remove([uri]);
  if (error) throw error;
  return { isSuccessful: true };
};

// 5. Secure Download (For 'chats' bucket)
const downloadFile = async ({ bucketName, uri }) => {
  const { data, error } = await supabase.storage
    .from(bucketName)
    .createSignedUrl(uri, 86400);
  if (error) throw error;
  return data?.signedUrl;
};
```

### 4. Fetching Image URLs

Storing only the `path` in the database is the standard. Use this conditional pattern to resolve the full URL for display:

```typescript
const resolveImageUrl = async (bucketName: BucketName, uri: string) => {
  if (bucketName === BucketName.CHATS) {
    // Private: Requires a signed URL or base64 download
    return await handleUpload.downloadFile({
      uri,
      bucketName,
      base64: false, // Set to true if raw data is needed
    });
  } else {
    // Public: Generate direct public link instantly
    return supabase.storage.from(bucketName).getPublicUrl(uri).data.publicUrl;
  }
};
```

> [!TIP]
> Public URLs are deterministic and generated client-side without a database call, making them extremely fast for lists and feeds.

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
