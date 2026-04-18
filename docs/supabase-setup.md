# Supabase Core Infrastructure

LinqUp uses **Supabase** as its unified backend provider, leveraging the full power of PostgreSQL, Realtime, and Edge Functions to handle complex dating and social logic entirely on the server side.

---

## 🏗️ Database Architecture

The system is built on a relational PostgreSQL schema with strict **Row Level Security (RLS)** to ensure user privacy and data integrity.

### 👥 User & Profiles
*   **`profiles`**: Extends the default `auth.users`. Stores core dating data such as `elo_rating`, `gender`, `sexual_orientation`, and `interests`.
*   **`user_preference_filters`**: Stores the user's discovery preferences (age range, distance, etc.) and calculated metrics like `avg_daily_swipes`.

### 💘 Recommendations & Matching
*   **`recommended_profiles`**: The central queue for the "Explore" deck. 
    *   `reaction`: Stores `'liked'`, `'rejected'`, or `NULL` (unseen).
    *   `profile_elo_at_time`: Snapshot of the recommended user's ELO when the recommendation was generated.
*   **`matches`**: Created when two users have a mutual `'liked'` reaction.
*   **`match_participants`**: Links users to a specific match.

### 🍽️ Vendors & Social
*   **`vendors`**: Information about restaurants and hangout spots.
*   **`bookings`**: Tracks reservations made by matches.
*   **`ledger`**: Handles wallet balances and internal transactions.

---

## ⚡ Automations & Triggers (The "Secret Sauce")

We offload heavy lifting to the database using PL/pgSQL functions and triggers.

### 1. ELO Rating System
When a user reacts to a recommendation, the `on_recommendation_reaction` trigger fires:
*   Updates the **ELO Rating** of the recommended profile based on the outcome (Like/Reject).
*   Uses a dynamic **K-factor** (lower for veteran users) to stabilize ratings over time.

### 2. Match Generation
The same trigger checks for a **Mutual Like**. If User B has already liked User A, a new record is automatically inserted into `matches` and `match_participants`.

### 3. Daily Recommendation Engine
*   **`generate_recommendations`**: A complex query that finds profiles matching a user's preferences, excluding already reacting users and blocked profiles.
*   **`run_nightly_recommendations`**: A wrapper that refreshes the queue for all active users.
*   **Cron Job**: Scheduled via `pg_cron` to run nightly at 2 AM UTC:
    ```sql
    select cron.schedule('nightly-recommendations', '0 2 * * *', 'select run_nightly_recommendations()');
    ```

---

## 🔒 Security (RLS)

Every table has RLS enabled. Policy patterns include:
*   **Profiles**: Public read for specific fields (name, photos), private write (`auth.uid() = id`).
*   **Matching**: Users can only see/update recommendations where `recommended_to = auth.uid()`.
*   **Matches**: Access granted only if `auth.uid()` is a participant in that match.

---

## 🚀 Edge Functions

Custom logic that requires external APIs or specialized processing is handled by Supabase Edge Functions:
*   **Payment Processing**: Integrating with Monnify for wallet funding.
*   **Image Processing**: Handling face verification and image optimization.
*   **Real-time Notifications**: Triggering push notifications via Expo SDK.
