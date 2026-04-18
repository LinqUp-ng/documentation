# Subscription Dashboard API Integration Guide
### RevenueCat + Apple App Store Connect + Google Play Developer API

> **Goal:** Build a custom dashboard that can read subscription data from both iOS and Android, and perform write operations including product/offering management.

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [RevenueCat API](#revenuecat-api)
3. [Apple App Store Connect API](#apple-app-store-connect-api)
4. [Google Play Developer API](#google-play-developer-api)
5. [Responsibility Matrix](#responsibility-matrix)
6. [Authentication Summary](#authentication-summary)
7. [Rate Limits](#rate-limits)
8. [Recommended Workflow for Product Creation](#recommended-workflow-for-product-creation)
9. [Gotchas & Caveats](#gotchas--caveats)

---

## Architecture Overview

RevenueCat sits as a **middle layer** between your app and the stores. It does not own products — those live in Apple and Google's systems. RevenueCat mirrors and abstracts them.

```
Your Dashboard
      │
      ├──► RevenueCat API         → Customers, entitlements, offerings,
      │    (api.revenuecat.com)     metrics, packages, promotional grants
      │
      ├──► App Store Connect API  → Create/modify iOS products, subscriptions,
      │    (api.appstoreconnect.   pricing, subscription groups
      │     apple.com)
      │
      └──► Google Play            → Create/modify Android in-app products,
           Developer API            subscriptions, base plans, offers
           (androidpublisher.
            googleapis.com)
```

**Rule of thumb:**
- Managing *customers and access* → RevenueCat API
- Managing *products in the stores* → Apple or Google APIs directly

---

## RevenueCat API

### Documentation Links
- **API v2 (current, actively developed):** https://www.revenuecat.com/docs/api-v2
- **API v1 (legacy but still required for some features):** https://www.revenuecat.com/docs/api-v1
- **Authentication / API Keys:** https://www.revenuecat.com/docs/projects/authentication
- **OpenAPI Spec (download):** https://www.revenuecat.com/docs/redocusaurus/plugin-redoc-0.yaml

> ⚠️ API v2 is still under active development. Some endpoints (like detailed subscriber info) still only exist in v1. You'll likely need to use **both versions** in parallel.

---

### Authentication

Two types of API keys exist:

| Key Type | Prefix | Use Case |
|---|---|---|
| Public | `appl_` / `goog_` | Client-side (SDK only) |
| Secret | `sk_` | Server-side, sensitive operations |
| OAuth Token | `atk_` | Third-party integrations, developer-level access |

**v1 auth header:**
```
Authorization: Bearer YOUR_SECRET_KEY
```

**v2 auth header (requires `Bearer` prefix per RFC 7235):**
```
Authorization: Bearer YOUR_SECRET_KEY
```

Secret keys can perform: deleting subscribers, granting entitlements, refunding purchases. Never expose them in client-side code or public repos.

---

### Base URLs

| Version | Base URL |
|---|---|
| v1 | `https://api.revenuecat.com/v1` |
| v2 | `https://api.revenuecat.com/v2` |

---

### What You Can Do via RevenueCat API

#### READ Operations

| Resource | Endpoint (v2) | Notes |
|---|---|---|
| Get customer info | `GET /projects/{project_id}/customers/{customer_id}` | Returns subscription state |
| List active entitlements | `GET /projects/{project_id}/customers/{customer_id}/active_entitlements` | Directly returns what's active |
| Get subscriber (v1) | `GET /v1/subscribers/{app_user_id}` | Includes full entitlement data |
| List products | `GET /projects/{project_id}/products` | Paginated |
| Get product | `GET /projects/{project_id}/products/{product_id}` | — |
| List offerings | `GET /projects/{project_id}/offerings` | Paginated |
| List entitlements | `GET /projects/{project_id}/entitlements` | Paginated |
| Overview metrics | `GET /projects/{project_id}/metrics/overview` | Revenue, subscribers, etc. |
| Chart data | `GET /projects/{project_id}/charts/{chart_name}` | Requires `charts_metrics:charts:read` |
| Audit logs | `GET /projects/{project_id}/audit_logs` | Requires `project_configuration:audit_logs:read` |

#### WRITE Operations

| Operation | Endpoint | Notes |
|---|---|---|
| Create product (RC level) | `POST /projects/{project_id}/products` | Maps to store product ID |
| Update product | `PATCH /projects/{project_id}/products/{product_id}` | — |
| Delete product | `DELETE /projects/{project_id}/products/{product_id}` | — |
| Create offering | `POST /projects/{project_id}/offerings` | — |
| Update offering | `PATCH /projects/{project_id}/offerings/{offering_id}` | — |
| Delete offering | `DELETE /projects/{project_id}/offerings/{offering_id}` | — |
| Create entitlement | `POST /projects/{project_id}/entitlements` | — |
| Grant promotional entitlement (v1) | `POST /v1/subscribers/{app_user_id}/entitlements/{entitlement_identifier}/promotional` | Secret key required |
| Revoke promotional entitlement (v1) | `POST /v1/subscribers/{app_user_id}/entitlements/{entitlement_identifier}/revoke_promotionals` | — |
| Create/update app | `POST/PATCH /projects/{project_id}/apps` | v2 only |
| Delete subscriber (v1) | `DELETE /v1/subscribers/{app_user_id}` | Permanent |
| Override offering for customer (v1) | `POST /v1/subscribers/{app_user_id}/offerings/{offering_uuid}/override` | — |
| Record purchase (v1) | `POST /v1/receipts` | iOS, Android, Stripe, Roku, Paddle |

#### Pagination

All list endpoints support pagination via:
```
GET /projects/{id}/products?limit=20&starting_after=LAST_ITEM_ID
```

Response shape:
```json
{
  "object": "list",
  "items": [...],
  "next_page": "URL_WITH_CURSOR",
  "url": "BASE_URL"
}
```

---

### RevenueCat Rate Limits (v2)

| Domain | Limit |
|---|---|
| Project Configuration | 60 req/min |
| Customer Information | 480 req/min |
| Charts & Metrics | 15 req/min |

Response headers on every request:
- `RevenueCat-Rate-Limit-Current-Usage`
- `RevenueCat-Rate-Limit-Current-Limit`
- `Retry-After` (on 429 responses)

---

## Apple App Store Connect API

### Documentation Links
- **Official overview:** https://developer.apple.com/app-store-connect/api/
- **Full API reference:** https://developer.apple.com/documentation/appstoreconnectapi
- **Getting started / key generation:** https://developer.apple.com/help/app-store-connect/get-started/app-store-connect-api/
- **App Store Server API** (for transaction verification): https://developer.apple.com/documentation/appstoreserverapi

---

### Authentication

Apple uses **JWT (JSON Web Token)** authentication — this is the biggest friction point compared to RevenueCat.

You need three things:
1. **Key ID** — from App Store Connect dashboard
2. **Issuer ID** — from App Store Connect dashboard
3. **.p8 private key file** — downloaded once (store it securely, cannot be re-downloaded)

**Generating the JWT:**
```javascript
import jwt from 'jsonwebtoken';
import fs from 'fs';

const privateKey = fs.readFileSync('./AuthKey_XXXXXXXXXX.p8');

const token = jwt.sign({}, privateKey, {
  algorithm: 'ES256',
  expiresIn: '20m', // Max 20 minutes
  issuer: 'YOUR_ISSUER_ID',
  audience: 'appstoreconnect-v1',
  header: {
    alg: 'ES256',
    kid: 'YOUR_KEY_ID',
    typ: 'JWT'
  }
});
```

**Request header:**
```
Authorization: Bearer YOUR_JWT_TOKEN
```

> ⚠️ The JWT expires after a maximum of 20 minutes. You need to regenerate it for each session or cache it and refresh before expiry.

**Key setup steps:**
1. Log into App Store Connect as Account Holder
2. Go to Users and Access → Integrations
3. Select App Store Connect API → Team Keys
4. Click Generate API Key
5. Assign appropriate role (Admin for full access)
6. Download the `.p8` file immediately — you only get one chance

---

### Base URL

```
https://api.appstoreconnect.apple.com/v1
```

---

### What You Can Do via App Store Connect API

#### Subscription Management

| Operation | Endpoint |
|---|---|
| List subscriptions | `GET /v1/apps/{appId}/subscriptions` |
| Create subscription | `POST /v1/subscriptions` |
| Read subscription | `GET /v1/subscriptions/{id}` |
| Modify subscription (name, review info) | `PATCH /v1/subscriptions/{id}` |
| Delete subscription | `DELETE /v1/subscriptions/{id}` |
| Create subscription group | `POST /v1/subscriptionGroups` |
| List subscription groups | `GET /v1/apps/{appId}/subscriptionGroups` |
| Set subscription pricing | `POST /v1/subscriptionPrices` |
| List available territories | `GET /v1/subscriptionPricePoints` |
| Submit subscription for review | `POST /v1/subscriptionAppStoreReviewScreenshots` |

#### In-App Purchase Management

| Operation | Endpoint |
|---|---|
| Create in-app purchase | `POST /v1/inAppPurchases` |
| Read in-app purchase | `GET /v1/inAppPurchases/{id}` |
| Modify in-app purchase | `PATCH /v1/inAppPurchases/{id}` |
| Delete in-app purchase | `DELETE /v1/inAppPurchases/{id}` |
| Set pricing | `POST /v1/inAppPurchasePriceSchedules` |

#### App & Analytics

| Operation | Endpoint |
|---|---|
| List apps | `GET /v1/apps` |
| Sales reports | `GET /v1/salesReports` |
| Subscription reports | `GET /v1/financeReports` |
| Customer reviews | `GET /v1/apps/{id}/customerReviews` |

> **Important limitation:** You can create and modify subscriptions, but **price changes and certain metadata changes may still require App Store review** before going live.

---

## Google Play Developer API

### Documentation Links
- **Official overview:** https://developers.google.com/android-publisher
- **Getting started:** https://developers.google.com/android-publisher/getting_started
- **Full REST API reference:** https://developers.google.com/android-publisher/api-ref/rest
- **Subscriptions v2 resource:** https://developers.google.com/android-publisher/api-ref/rest/v3/purchases.subscriptionsv2
- **In-app products resource:** https://developers.google.com/android-publisher/api-ref/rest/v3/inappproducts
- **Release notes:** https://developer.android.com/google/play/billing/play-developer-apis-release-notes

---

### Authentication

Google Play uses **OAuth 2.0 with a Service Account** (recommended for server-to-server calls).

**Setup steps:**
1. Go to Google Play Console → Setup → API access
2. Link to a Google Cloud Project
3. Create a Service Account in Google Cloud IAM
4. Grant the service account permissions in Play Console (Financial data, Manage orders, etc.)
5. Download the service account JSON key

**Getting an access token (Node.js example):**
```javascript
import { google } from 'googleapis';

const auth = new google.auth.GoogleAuth({
  keyFile: './service-account.json',
  scopes: ['https://www.googleapis.com/auth/androidpublisher']
});

const client = await auth.getClient();
const token = await client.getAccessToken();
```

**Request header:**
```
Authorization: Bearer ACCESS_TOKEN
```

---

### Base URL

```
https://androidpublisher.googleapis.com/androidpublisher/v3
```

---

### Two API Modes: Edits vs Immediate

> This is a critical concept unique to Google's API.

- **Publishing API** uses an **"edits" / transactional model** — you bundle changes into a draft edit, then commit them all at once. Changes don't take effect until committed.
- **Subscriptions & In-App Purchases API** takes **immediate effect** — no edits model needed.

---

### What You Can Do via Google Play Developer API

#### In-App Products (One-time purchases)

| Operation | Endpoint |
|---|---|
| List in-app products | `GET /applications/{packageName}/inappproducts` |
| Get in-app product | `GET /applications/{packageName}/inappproducts/{sku}` |
| Create in-app product | `POST /applications/{packageName}/inappproducts` |
| Update in-app product | `PUT /applications/{packageName}/inappproducts/{sku}` |
| Patch in-app product | `PATCH /applications/{packageName}/inappproducts/{sku}` |
| Delete in-app product | `DELETE /applications/{packageName}/inappproducts/{sku}` |
| Batch get | `GET /applications/{packageName}/inappproducts:batchGet` |
| Batch update | `POST /applications/{packageName}/inappproducts:batchUpdate` |

#### Subscriptions

| Operation | Endpoint |
|---|---|
| List subscriptions | `GET /applications/{packageName}/subscriptions` |
| Get subscription | `GET /applications/{packageName}/subscriptions/{productId}` |
| Create subscription | `POST /applications/{packageName}/subscriptions` |
| Patch subscription | `PATCH /applications/{packageName}/subscriptions/{productId}` |
| Delete subscription | `DELETE /applications/{packageName}/subscriptions/{productId}` |
| Batch get | `POST /applications/{packageName}/subscriptions:batchGet` |
| Batch update | `POST /applications/{packageName}/subscriptions:batchUpdate` |

#### Base Plans & Offers (for subscriptions)

| Operation | Endpoint |
|---|---|
| List base plans | `GET /applications/{packageName}/subscriptions/{productId}/basePlans` |
| Create base plan | `POST /applications/{packageName}/subscriptions/{productId}/basePlans` |
| Activate base plan | `POST /applications/{packageName}/subscriptions/{productId}/basePlans/{basePlanId}:activate` |
| Deactivate base plan | `POST /applications/{packageName}/subscriptions/{productId}/basePlans/{basePlanId}:deactivate` |
| Create offer | `POST /applications/{packageName}/subscriptions/{productId}/basePlans/{basePlanId}/offers` |
| List offers | `GET /applications/{packageName}/subscriptions/{productId}/basePlans/{basePlanId}/offers` |
| Activate offer | `POST /...offers/{offerId}:activate` |

#### Purchase Verification & Management

| Operation | Endpoint |
|---|---|
| Get subscription purchase (v2) | `GET /applications/{packageName}/purchases/subscriptionsv2/tokens/{token}` |
| Cancel subscription | `POST /applications/{packageName}/purchases/subscriptions/{subscriptionId}/tokens/{token}:cancel` |
| Defer subscription billing | `POST /applications/{packageName}/purchases/subscriptionsv2/{token}:defer` |
| Revoke subscription | `POST /applications/{packageName}/purchases/subscriptions/{subscriptionId}/tokens/{token}:revoke` |
| Acknowledge purchase | `POST /applications/{packageName}/purchases/subscriptions/{subscriptionId}/tokens/{token}:acknowledge` |
| Verify one-time product purchase | `GET /applications/{packageName}/purchases/products/{productId}/tokens/{token}` |

#### Real-Time Developer Notifications (RTDNs)

Google pushes subscription events via **Pub/Sub** — set this up in Play Console to receive webhook-like events:
- `SUBSCRIPTION_PURCHASED`
- `SUBSCRIPTION_RENEWED`
- `SUBSCRIPTION_CANCELED`
- `SUBSCRIPTION_EXPIRED`
- `SUBSCRIPTION_PRICE_STEP_UP_CONSENT_UPDATED` (new)

**Setup:** Play Console → Monetization → Monetization setup → Real-time developer notifications

---

## Responsibility Matrix

| Task | RevenueCat API | App Store Connect API | Google Play API |
|---|---|---|---|
| Fetch subscriber info (iOS + Android) | ✅ Unified | ❌ | ❌ |
| Check if user has active subscription | ✅ | ❌ | ❌ |
| Grant/revoke promotional access | ✅ | ❌ | ❌ |
| Manage Offerings & Packages | ✅ | ❌ | ❌ |
| Revenue/metrics dashboard | ✅ | ✅ | ✅ (Reporting API) |
| Create iOS subscription product | ❌ | ✅ | ❌ |
| Modify iOS subscription price | ❌ | ✅ | ❌ |
| Create Android subscription product | ❌ | ❌ | ✅ |
| Create Android base plans & offers | ❌ | ❌ | ✅ |
| Cancel a subscription | ✅ (entitlement revoke) | Limited | ✅ |
| Refund a purchase | ✅ (v1) | Via App Store Server API | ✅ |
| Receive real-time events | ✅ (Webhooks) | ✅ (Webhooks, new 2025) | ✅ (Pub/Sub RTDNs) |
| Sync store products into RevenueCat | N/A | → Import to RC after creating | → Import to RC after creating |

---

## Authentication Summary

| API | Auth Method | Token Type | Expiry |
|---|---|---|---|
| RevenueCat v1 & v2 | API Key | Static secret key (`sk_...`) | Never (until revoked) |
| RevenueCat v2 (OAuth) | OAuth 2.0 | Access token (`atk_...`) | Session-based |
| App Store Connect | JWT (ES256) | Signed JWT from `.p8` key | Max 20 minutes |
| Google Play | OAuth 2.0 (Service Account) | Bearer access token | ~1 hour |

---

## Rate Limits

### RevenueCat
See per-domain limits in the RevenueCat section above. Use the `Retry-After` header when hitting 429s.

### App Store Connect
Apple does not publish exact rate limit numbers but enforces limits. Best practices:
- Cache responses aggressively
- Don't poll — use webhooks where available
- Exponential backoff on errors

### Google Play
Google enforces a daily quota per project. Recommendations:
- **Cache purchase details server-side** — do not call the API on every user action
- **Store subscription expiry locally** and only re-query at/near expiry
- **Do not publish updates more than once per day** for alpha/beta tracks
- Request quota increases via Google Cloud Console if needed

---

## Recommended Workflow for Product Creation

### Creating a new iOS subscription from your dashboard:

```
1. Call App Store Connect API
   POST /v1/subscriptions
   Body: { name, productId, subscriptionGroupId, reviewNote, ... }

2. Set pricing
   POST /v1/subscriptionPrices
   Body: { subscriptionId, territory, price, ... }

3. Submit for review (if required)
   POST /v1/subscriptionAppStoreReviewScreenshots

4. After approval, import into RevenueCat
   POST https://api.revenuecat.com/v2/projects/{id}/products
   Body: { store_identifier: "your_product_id", app_id: "rc_app_id" }

5. Attach to Offering/Package in RevenueCat
   POST /v2/projects/{id}/offerings/{offering_id}/packages
```

### Creating a new Android subscription from your dashboard:

```
1. Call Google Play Developer API
   POST /applications/{packageName}/subscriptions
   Body: { productId, listings: [...], basePlans: [...] }

2. Create base plan
   POST /applications/{packageName}/subscriptions/{productId}/basePlans
   Body: { billingPeriodDuration, regionalConfigs: [{ regionCode, price, ... }] }

3. Activate the base plan
   POST /applications/{packageName}/subscriptions/{productId}/basePlans/{id}:activate

4. Import into RevenueCat
   POST https://api.revenuecat.com/v2/projects/{id}/products
   Body: { store_identifier: "your_product_id", app_id: "rc_app_id" }

5. Attach to Offering/Package in RevenueCat
```

---

## Gotchas & Caveats

### RevenueCat
- **v2 is incomplete** — the subscriber/entitlement detail you need for some flows is still only in v1. Community members have confirmed that `active_entitlements` may return empty in v2 while v1 returns them correctly.
- **Secret keys must never be in client-side code.** All write operations from your dashboard should go through your own backend server, which then calls RevenueCat.
- **Creating a "product" in RevenueCat** only creates a RevenueCat-level mapping. The actual product must already exist in the App Store or Play Store first.
- **Sandbox vs Production** — some v2 endpoints have an `environment` field; others (like `active_entitlements`) may default to sandbox. Test carefully.

### Apple App Store Connect
- **The `.p8` key file can only be downloaded once.** If lost, you must revoke and generate a new key.
- **JWTs expire in 20 minutes max.** Build token refresh logic into your dashboard.
- **Not all product changes are instant.** Price changes and some metadata changes require App Store review.
- **App Store Connect API is a JSON:API spec** — the response structure is nested and relationship-based, which can be confusing. Use an OpenAPI-generated client library where possible.
- **Useful trick:** To find which endpoint maps to a UI action, open App Store Connect in your browser and inspect the network traffic in DevTools.

### Google Play
- **"Edits" model vs Immediate** — do not confuse the two. Publishing API changes use edits (must be committed); purchase/subscription status API calls take effect immediately.
- **Service account permissions** must be explicitly granted in Play Console, not just in Google Cloud IAM. Both places need to be configured.
- **Subscription states are complex:** pending, active, paused, grace period, on hold, canceled, expired — your dashboard UI needs to handle all of these gracefully.
- **Always cache** — Google strongly recommends server-side caching of purchase tokens and expiry dates. Do not hit their API on every user request.
- **New `subscriptionsv2` API** is the current recommended version. The older `purchases.subscriptions` endpoints are being deprecated.
- **Real-time events via Pub/Sub** — if you want to keep your dashboard live without polling, set up Google Cloud Pub/Sub and configure RTDNs in Play Console.

---

## Useful Libraries & Tools

| Library | Language | Purpose |
|---|---|---|
| `@revenuecat/purchases-js` | JavaScript | RevenueCat Web SDK |
| `revenuecat` (npm) | Node.js | Unofficial RC REST client |
| `googleapis` | Node.js | Official Google API client (includes Play) |
| `app-store-connect-api` | Node.js | App Store Connect API wrapper |
| `appstoreconnect` | Python | Python wrapper for ASC API |
| `app-store-connect-swift-sdk` | Swift | Swift types for ASC API |

---

## Key Reference Links (All in One Place)

| Resource | URL |
|---|---|
| RevenueCat API v2 Docs | https://www.revenuecat.com/docs/api-v2 |
| RevenueCat API v1 Docs | https://www.revenuecat.com/docs/api-v1 |
| RevenueCat OpenAPI Spec | https://www.revenuecat.com/docs/redocusaurus/plugin-redoc-0.yaml |
| RevenueCat Authentication | https://www.revenuecat.com/docs/projects/authentication |
| RevenueCat Community | https://community.revenuecat.com |
| App Store Connect API Docs | https://developer.apple.com/documentation/appstoreconnectapi |
| App Store Connect API Overview | https://developer.apple.com/app-store-connect/api/ |
| App Store Connect Key Setup | https://developer.apple.com/help/app-store-connect/get-started/app-store-connect-api/ |
| App Store Server API | https://developer.apple.com/documentation/appstoreserverapi |
| Google Play Developer API | https://developers.google.com/android-publisher |
| Google Play API Reference | https://developers.google.com/android-publisher/api-ref/rest |
| Google Play Subscriptions v2 | https://developers.google.com/android-publisher/api-ref/rest/v3/purchases.subscriptionsv2 |
| Google Play Getting Started | https://developers.google.com/android-publisher/getting_started |
| Google Play Release Notes | https://developer.android.com/google/play/billing/play-developer-apis-release-notes |
| Google Play Billing Subscriptions Guide | https://developer.android.com/google/play/billing/subscriptions |
