# Supabase Setup & Configuration

This guide covers the technical initialization of the Supabase client and the core infrastructure configurations for LinqUp.

---

## 🏗️ Client Initialization

We enforce strict typing across the application by using the generated `Database` types. These types allow for compile-time validation of your queries and table relations.

> [!IMPORTANT]
> **Security Reminder**: Never expose the full `Database` type file in public documentation. Always reference it relative to your project structure (e.g., `@/types/supabase.ts`).

```typescript
import { createClient } from '@supabase/supabase-js';
import { Database } from '@/types/supabase';

const supabaseUrl = process.env.EXPO_PUBLIC_SUPABASE_URL!;
const supabaseAnonKey = process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY!;

export const supabase = createClient<Database>(supabaseUrl, supabaseAnonKey);
```

---

## 💾 Persistent Storage

The storage engine for auth tokens and local state differs based on the target platform. You must ensure the correct library is used to prevent session loss.

### 📱 Mobile (React Native / Expo)
To maintain user sessions between app restarts, you must use `AsyncStorage`.
```bash
npx expo install @react-native-async-storage/async-storage
```

### 🌐 Web (Next.js / React)
For web deployments, use a browser-compatible library. We recommend **standard browser storage** or a cookie-based approach for server-side compatibility. 
> [!WARNING]
> Do NOT use `@react-native-async-storage/async-storage` on the web; it is not compatible with standard browser environments.

---

## ⚡ Core Automations

LinqUp leverages PostgreSQL triggers to handle high-frequency events without client-side intervention:
- **ELO Rating**: Updated automatically via the `on_recommendation_reaction` trigger.
- **Match Creation**: Handled by the database upon detecting mutual likes.
- **JWT Claims Sync**: Custom roles (`admin`, `vendor`, `user`) are automatically injected into the Auth JWT via database triggers on the `public.users` table.
