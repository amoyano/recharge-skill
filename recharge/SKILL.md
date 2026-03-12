---
name: recharge
description: Complete technical skill for the Recharge Subscriptions platform (Shopify). Covers four areas — Admin API, Theme Engine, JavaScript SDK, and Webhooks. Use when working with Recharge subscription management, customer portal customization, headless storefront integrations, API calls, webhook management, charge processing, billing issues, or troubleshooting. Triggers — "recharge", "recharge API", "recharge subscriptions", "resync subscriptions", "fix recharge", "reinitialize webhooks", "refresh billing", "subscription not working", "recharge is broken", "recharge theme engine", "customer portal", "storefront SDK", "recharge SDK", "recharge JavaScript", "recurring billing", "failed charges", "recharge webhook", "subscription cancelled", "recharge headless", "recharge hydrogen".
---

# Recharge Subscriptions — Complete Skill

Recharge is a subscription management platform for Shopify merchants enabling recurring billing, subscription management, and automated repeat orders.

## Quick Reference

| Setting | Value |
|---------|-------|
| **API Base URL** | `https://api.rechargeapps.com` |
| **Auth header** | `X-Recharge-Access-Token: <token>` |
| **API Version** | `2021-11` (header: `X-Recharge-Version`) |
| **SDK Package** | `@rechargeapps/storefront-client` |
| **CDN Base** | `https://static.rechargecdn.com/store/{identifier}` |
| **Portal Base** | `/tools/recurring/portal/{customer_hash}/` |

## Platforms

| Tag | Platform | Checkout | Customer Portal |
|-----|----------|----------|----------------|
| `RCS` | Shopify | Recharge-hosted | Theme Engine / Affinity |
| `SCI` | Shopify | Shopify-hosted | Affinity |
| `Custom` | API-first / headless | API / custom | Storefront SDK |
| `BigCommerce` | BigCommerce | Recharge-hosted | Limited |

## Architecture Layers

```
┌─────────────────────────────────────────────────────┐
│                  Merchant Admin                      │
│            (Configure store, products, plans)        │
├─────────────────────────────────────────────────────┤
│                                                      │
│  ┌──────────┐  ┌──────────┐  ┌───────────────────┐ │
│  │ Admin API │  │ Webhooks │  │     CDN           │ │
│  │ (Server)  │  │ (Events) │  │ (Product data)    │ │
│  └────┬─────┘  └────┬─────┘  └────────┬──────────┘ │
│       │              │                  │            │
│  ┌────┴──────────────┴──────────────────┴─────────┐ │
│  │          Integration Layer                      │ │
│  ├─────────────────┬───────────────────────────────┤ │
│  │  Theme Engine   │    Storefront JS SDK          │ │
│  │  (Shopify       │    (@rechargeapps/            │ │
│  │   portal)       │     storefront-client)        │ │
│  └─────────────────┴───────────────────────────────┘ │
│                                                      │
│  ┌─────────────────────────────────────────────────┐ │
│  │         Customer-Facing Storefront              │ │
│  │    (Shopify theme / Hydrogen / Custom)          │ │
│  └─────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────┘
```

---

## Common Operations (Quick Recipes)

### 1. Re-sync Subscription State
```http
GET /subscriptions/{id}
GET /subscriptions?customer_id={id}&limit=250
PUT /subscriptions/{id}   # Force update to trigger state sync
POST /subscriptions/{id}/activate  # Reactivate cancelled subscription
POST /subscriptions/{id}/set_next_charge_date  # Fix next charge date
```

### 2. Reinitialize / Reset Webhooks
```http
GET /webhooks                    # List current webhooks
DELETE /webhooks/{id}            # Remove stale webhook
POST /webhooks                   # Re-register
  body: { "address": "https://...", "topic": "subscription/created", "included_objects": ["customer"] }
POST /webhooks/{id}/test         # Verify webhook fires
```

### 3. Refresh Billing / Retry Failed Charge
```http
GET /charges?status=ERROR&limit=250      # Find failed charges
POST /charges/{id}/process               # Retry a failed charge
POST /charges/{id}/unskip               # Unskip a skipped charge
POST /charges/{id}/capture_payment      # Capture uncaptured charge
POST /charges/{id}/apply_discount       # Apply discount code
```

### 4. Reconnect Payment Method
```http
GET /payment_methods?customer_id={id}   # List payment methods
PUT /payment_methods/{id}               # Update payment method
POST /notifications/send_email          # Prompt customer to update
```

### 5. Bulk Re-sync via Async Batch
```http
POST /async_batches
  body: { "batch_type": "subscription_update" }
POST /async_batches/{id}/process
GET  /async_batches/{id}                # Check status
```

### 6. Merge / Consolidate Addresses
```http
POST /addresses/merge
  body: { "target_address": {"id": 123}, "source_addresses": [{"id": 456}], "delete_source_addresses": true }
```

### 7. Theme Engine — Fetch Customer Portal Data
```javascript
const schema = JSON.stringify({
  customer: {}, subscriptions: [], addresses: [],
  payment_methods: [], orders: [], settings: {}, store: {}
});
const url = `/portal/${hash}/request_objects?token=${token}&schema=${schema}`;
const data = await fetch(url).then(r => r.json());
```

### 8. JavaScript SDK — Quick Start
```typescript
import { initRecharge, loginShopifyAppProxy, listSubscriptions } from '@rechargeapps/storefront-client';

initRecharge({ storeIdentifier: 'store.myshopify.com', storefrontAccessToken: 'strfnt_...' });
const session = await loginShopifyAppProxy();
const subs = await listSubscriptions(session, { limit: 25 });
```

---

## Skill Reference Files

Each area has a dedicated, comprehensive reference file:

| Topic | File | When to Read |
|-------|------|-------------|
| **Admin API** (all endpoints, objects, scopes, pagination, sorting) | `references/api_reference.md` | Building server-side integrations, managing subscriptions/charges/customers via API, understanding request/response formats |
| **Theme Engine** (portal routes, Jinja templates, request objects, Affinity) | `references/theme_engine.md` | Customizing customer portal, building portal pages, working with Jinja templates, fetching data via request_objects |
| **JavaScript SDK** (storefront-client, auth, CDN, bundles, all methods) | `references/javascript_sdk.md` | Building headless storefronts, Hydrogen integration, client-side subscription management, widget building |
| **Webhooks** (all topics, payloads, included_objects, reset procedures) | `references/webhooks.md` | Setting up event listeners, debugging webhook delivery, building event-driven integrations |
| **Troubleshooting** (diagnostic tree, common fixes, platform notes) | `references/troubleshooting.md` | Diagnosing issues, fixing failed charges, resolving sync problems, debugging auth errors |

### When to Use Which Reference

- **"How do I call the Recharge API?"** → `api_reference.md`
- **"How do I customize the customer portal?"** → `theme_engine.md`
- **"How do I build a headless subscription widget?"** → `javascript_sdk.md`
- **"How do I listen for subscription events?"** → `webhooks.md`
- **"Something is broken with Recharge"** → `troubleshooting.md`
- **"How do I set up Recharge from scratch?"** → `api_reference.md` (auth + store) + `javascript_sdk.md` (SDK init)
