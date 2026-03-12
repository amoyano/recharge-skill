---
name: recharge
description: 'Technical skill for the Recharge Subscriptions platform (Shopify). Use when working with Recharge subscription management - re-syncing data, refreshing billing state, reinitializing webhooks, reconnecting APIs, retrying failed charges, managing recurring orders, payment methods, and resolving subscription processing issues.'
---

# Recharge Subscriptions

Recharge is a subscription management platform for Shopify merchants enabling recurring billing, subscription management, and automated repeat orders.

**API Base URL:** `https://api.rechargeapps.com`
**Auth header:** `X-Recharge-Access-Token: <token>`
**Docs:** https://developer.rechargepayments.com/2021-11

## Platforms

| Tag | Platform |
|-----|----------|
| `RCS` | Recharge hosted on Shopify |
| `SCI` | Shopify-hosted checkout |
| `Custom` | API-first / headless |

## Common Recharge/Refresh Operations

### 1. Re-sync Subscription State
```http
GET /subscriptions/{id}
GET /subscriptions?customer_id={id}&limit=250
PUT /subscriptions/{id}   # Force update to trigger state sync
POST /subscriptions/{id}/activate  # Reactivate cancelled subscription
```

### 2. Reinitialize / Reset Webhooks
```http
GET /webhooks                    # List current webhooks
DELETE /webhooks/{id}            # Remove stale webhook
POST /webhooks                   # Re-register
  body: { "address": "https://...", "topic": "subscription/created" }
POST /webhooks/{id}/test         # Verify webhook fires
```

### 3. Refresh Billing / Retry Failed Charge
```http
GET /charges?status=error&limit=50   # Find failed charges
POST /charges/{id}/process           # Retry a failed charge
POST /charges/{id}/unskip            # Unskip a skipped charge
POST /charges/{id}/capture           # Capture uncaptured charge
```

### 4. Reconnect Payment Method
```http
GET /payment_methods?customer_id={id}   # List payment methods
PUT /payment_methods/{id}               # Update payment method
POST /customers/{id}/send_payment_update_email  # Prompt customer to update
```

### 5. Bulk Re-sync via Async Batch
```http
POST /async_batches
  body: { "batch_type": "subscription_update" }
POST /async_batches/{id}/process
```

### 6. Merge / Consolidate Addresses
```http
POST /addresses/merge
  body: { "next_address_id": 123, "previous_address_ids": [456, 789] }
```

## Resources

| Topic | File |
|-------|------|
| All Admin API endpoints | `references/api_reference.md` |
| Webhook topics & payloads | `references/webhooks.md` |
| Common errors & fixes | `references/troubleshooting.md` |
| Theme Engine & Storefront SDK | `references/theme_and_sdk.md` |
