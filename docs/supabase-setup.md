# Supabase Infrastructure & configuration

LinqUp uses **Supabase** as its unified backend provider, leveraging PostgreSQL, Realtime, and Edge Functions to handle complex social logic entirely on the server side.

---

## 🏗️ Client Setup & Configuration

### Initialization
When creating the Supabase client, we enforce strict typing using the generated `Database` types (typically exported to `@/types/supabase.ts`).

> [!IMPORTANT]
> For security reasons, never include the full `Database` type file in your public documentation or commit it to unprotected public repositories.

```typescript
import { createClient } from '@supabase/supabase-js';
import { Database } from '@/types/supabase';

export const supabase = createClient<Database>(
  process.env.EXPO_PUBLIC_SUPABASE_URL!,
  process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY!
);
```

### Persistent Storage
The app uses different storage engines depending on the platform:
- **Mobile (React Native/Expo)**: Must use `AsyncStorage` from `@react-native-async-storage/async-storage`.
- **Web (Next.js/React)**: Must use a web-compatible library (like `localStorage` or `cookie-storage`) as `AsyncStorage` is not available in standard browser environments.

---

## 🔐 Authentication & Roles

### User Lifecycle
1. **Login**: Standard email/password or OTP authentication.
2. **Onboarding**: Users must complete their profile before accessing the main app.
3. **Roles**: Managed via custom JWT claims synced from the database.

### Administrative Workflows
Administrators are managed via specialized Edge Functions to ensure high security and relational integrity.

| Function | Requirement | Action |
| :--- | :--- | :--- |
| `add-admin` | **Super Admin** only | Creates Auth user, Public record, and sends invitation email via Resend. |
| `deactivate-admin` | **Super Admin** only | Toggles the `is_deactivated` flag. Instantly updates JWT claims to block access. |

---

## 📁 File Storage (Supabase Storage)

### Bucket Configuration
We use multiple buckets to segregate content by sensitivity and usage.

| Bucket | Access | Purpose |
| :--- | :--- | :--- |
| `profiles` | Public | User profile photos and gallery images. |
| `restaurants` | Public | Restaurant cover photos and promotional images. |
| `businesses` | Public | CAC documents and tax IDs (Read-only for Admins). |
| `chat` | **Private** | Images sent in direct messages. Requires signed URLs. |
| `system` | Private | Internal assets, CSV seeds, and setup files. |

### Retrieving File URLs
Depending on the bucket privacy, use one of these two methods located in `api/handleUpload.ts`:

**1. Public URL (Fast / Shared)**
Used for profile pictures where privacy is less critical.
```typescript
const { data } = supabase.storage.from('profiles').getPublicUrl(path);
```

**2. Signed URL (Secure / Private)**
Used for the `chat` bucket. Generates a temporary link that expires.
```typescript
const { data, error } = await supabase.storage
    .from('chat')
    .createSignedUrl(path, 3600); // Valid for 1 hour
```

---

## ⚡ Automations & Triggers
We offload heavy lifting to the database using PL/pgSQL functions and triggers.
- **ELO Rating System**: Updates dynamically after every swipe interaction.
- **Match Generation**: Automatically detects mutual likes and creates match records.
- **JWT Claims Sync**: Ensures the frontend always knows the user's `type` (admin/user/vendor) without extra API calls.
