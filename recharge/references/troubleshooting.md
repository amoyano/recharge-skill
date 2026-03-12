# Recharge Troubleshooting & Common Fixes

Complete troubleshooting guide for Recharge subscription issues. Use this skill when diagnosing Recharge integration problems, fixing failed charges, resolving sync issues, debugging authentication errors, or handling customer portal problems.

**Docs:** https://docs.getrecharge.com/ | https://developer.rechargepayments.com/2021-11

## Diagnostic Decision Tree

```
Recharge issue reported
├── Webhooks not firing?         → See "Webhook Issues"
├── Charges failing/erroring?    → See "Charge Failures"
├── Subscriptions out of sync?   → See "Subscription Sync"
├── Payment method issues?       → See "Payment Problems"
├── API auth errors?             → See "Authentication Errors"
├── Orders not generating?       → See "Order Generation Issues"
├── Address/merge problems?      → See "Address Issues"
├── Portal not loading?          → See "Theme Portal Issues"
├── SDK/Storefront errors?       → See "SDK Issues"
└── Rate limiting?               → See "Rate Limiting"
```

---

## Webhook Issues

**Symptom:** Events not received, webhooks silent

```bash
# Step 1: List all registered webhooks
GET /webhooks

# Step 2: Test a specific webhook
POST /webhooks/{id}/test

# Step 3: If stale/broken, delete and re-register
DELETE /webhooks/{id}
POST /webhooks
{
  "address": "https://your-endpoint.com/recharge",
  "topic": "subscription/updated",
  "included_objects": ["customer", "metafields"]
}
```

**Common causes:**
- Endpoint URL changed (store migration, new domain)
- Webhook registered to old staging URL
- Duplicate webhooks causing conflicts
- Missing `included_objects` for expected payload fields
- Endpoint returning non-2xx status codes
- SSL certificate issues on receiving endpoint
- IP allowlisting blocking Recharge servers

**Full reset procedure:** See `references/webhooks.md` for programmatic reset.

---

## Charge Failures

**Symptom:** `charge/failed` or `charge/max_retries_reached` events

```bash
# Find all failed charges
GET /charges?status=ERROR&limit=250

# Check charge details and activities
GET /charges/{id}?include=charge_activities,customer,payment_methods

# Retry a specific charge
POST /charges/{id}/process

# Unskip a skipped charge
POST /charges/{id}/unskip
{"purchase_item_ids": [123, 456]}
```

**Charge statuses:**

| Status | Meaning | Action |
|--------|---------|--------|
| `QUEUED` | Scheduled, not yet processed | Wait or process manually |
| `SKIPPED` | Manually/automatically skipped | Unskip if needed |
| `ERROR` | Failed, can be retried | Process to retry |
| `SUCCESS` | Processed successfully | No action needed |
| `REFUNDED` | Refunded after success | No action needed |
| `PARTIALLY_REFUNDED` | Partially refunded | Review refund amount |
| `UNCAPTURED` | Authorized but not captured | Capture payment |

**Common failure reasons:**
- Expired credit card → Send payment update email: `POST /notifications/send_email`
- Insufficient funds → Retry after delay
- Payment processor timeout → Retry once
- Invalid billing address → Update address first
- Declined by bank → Customer needs to contact bank

**Bulk retry pattern:**
```javascript
const { charges } = await fetch('/charges?status=ERROR&limit=250', { headers }).then(r => r.json());
for (const charge of charges) {
  try {
    await fetch(`/charges/${charge.id}/process`, { method: 'POST', headers });
    console.log(`Retried charge ${charge.id}`);
  } catch (err) {
    console.error(`Failed to retry charge ${charge.id}:`, err.message);
  }
  await new Promise(r => setTimeout(r, 500)); // rate limit buffer
}
```

---

## Subscription Sync

**Symptom:** Subscription state stale or out of sync with Shopify

```bash
# Force a sync by reading + updating with same data
GET /subscriptions/{id}
PUT /subscriptions/{id}
{"status": "ACTIVE"}

# Reactivate a cancelled subscription
POST /subscriptions/{id}/activate

# Check full subscription with all includes
GET /subscriptions/{id}?include=address,customer,metafields,charge_activities

# Verify the subscription's next charge date
GET /subscriptions/{id}
# If wrong:
POST /subscriptions/{id}/set_next_charge_date
{"date": "2024-03-01"}
```

**Common sync issues:**
- Subscription shows cancelled but should be active → `POST /subscriptions/{id}/activate`
- Next charge date is wrong → `POST /subscriptions/{id}/set_next_charge_date`
- Subscription on wrong address → `POST /subscriptions/{id}/change_address {"address_id": NEW_ID}`
- Quantity mismatch → `PUT /subscriptions/{id} {"quantity": CORRECT}`
- Product/variant changed in Shopify but not in Recharge → Update via API

**Bulk sync via async batches:**
```bash
POST /async_batches
{"batch_type": "subscription_update"}

POST /async_batches/{id}/process

# Monitor
GET /async_batches/{id}
```

---

## Payment Problems

**Symptom:** Payment method invalid, expired, or missing

```bash
# Get customer's payment methods
GET /payment_methods?customer_id={id}

# Update payment method
PUT /payment_methods/{id}
{
  "payment_token": "new_token",
  "payment_type": "CREDIT_CARD"
}

# Send update email to customer
POST /notifications/send_email
{
  "type": "payment_update",
  "template_vars": {"customer_id": 123}
}
```

**Payment types:** `CREDIT_CARD`, `PAYPAL`, `APPLE_PAY`, `GOOGLE_PAY`, `SEPA_DEBIT`

**Dunning indicators:**
- `customer.has_payment_method_in_dunning` = true → active dunning
- `charge.status` = `ERROR` → payment failed
- `charge.error_type` → specific failure reason

---

## Authentication Errors

**Symptom:** `401 Unauthorized` or `403 Forbidden`

**Checklist:**
1. Verify token: `GET /store` should return store info
2. Check header format: `X-Recharge-Access-Token: your_api_token`
3. Check version header: `X-Recharge-Version: 2021-11`
4. Verify required scopes for the operation
5. Confirm correct store — different tokens for different stores
6. Check if store is still installed (403 = uninstalled store)

**Common scopes needed:**

| Operation | Scope |
|-----------|-------|
| Read subscriptions | `read_subscriptions` |
| Modify subscriptions | `write_subscriptions` |
| Process charges | `write_orders` |
| Customer data | `read_customers` / `write_customers` |
| Webhooks | `read_webhooks` / `write_webhooks` |
| Payment methods | `read_payment_methods` / `write_payment_methods` |
| Products | `read_products` |
| Store info | `read_store` |

**SDK auth issues:**
- `Invalid Recharge storefront access token` → Check `storefrontAccessToken` starts with `strfnt_`
- `Customer does not exist in Recharge` → Customer hasn't subscribed yet
- `Failed to validate customer with Shopify` → Shopify customer token expired
- Session expired (1 hour) → Re-authenticate with login function

---

## Order Generation Issues

**Symptom:** Orders not being created from subscriptions

```bash
# Check upcoming orders
GET /orders?status=QUEUED&limit=50

# Check customer delivery schedule
GET /customers/{id}/delivery_schedule

# Verify subscription next charge date
GET /subscriptions/{id}

# Manually set next charge date
POST /subscriptions/{id}/set_next_charge_date
{"date": "2024-01-15"}
```

**Checklist:**
- Is subscription status `ACTIVE`?
- Is `next_charge_scheduled_at` in the past or future?
- Is the payment method valid?
- Has the charge been created but failed?
- Check `charges?customer_id={id}&status=ERROR`

---

## Address / Merge Issues

**Symptom:** Subscriptions split across multiple addresses, duplicate data

```bash
# List customer addresses
GET /addresses?customer_id={id}

# Merge addresses (max 10 sources)
POST /addresses/merge
{
  "target_address": {"id": 123},
  "source_addresses": [{"id": 456}, {"id": 789}],
  "delete_source_addresses": true
}
```

**Rules:**
- Cannot merge if addresses have different `presentment_currency`
- Maximum 10 source addresses per merge
- Only addresses with no active subscriptions can be deleted independently

---

## Theme Portal Issues (Customer-Facing)

**Symptom:** Customer portal not loading, data not displaying, JS errors

```javascript
// Re-fetch customer state
const schema = JSON.stringify({
  customer: {},
  subscriptions: [],
  payment_methods: [],
  addresses: [],
  settings: {},
  store: {}
});
const url = `/portal/${customerHash}/request_objects?token=${token}&schema=${schema}`;
const data = await fetch(url).then(r => r.json());
```

**Common causes:**
- Token expired → Customer needs to re-login
- Invalid `customerHash` in URL
- `window.customerToken` not set
- Shopify proxy not configured correctly for embedded portal
- Custom JS errors in theme template
- Request objects schema has syntax errors

**Debug steps:**
1. Check browser console for JS errors
2. Verify `window.customerToken` exists
3. Test `request_objects` endpoint directly
4. Check network tab for failed requests
5. Verify Shopify app proxy settings

---

## SDK Issues

**Symptom:** JavaScript SDK errors, failed API calls

| Error | Cause | Fix |
|-------|-------|-----|
| `initRecharge not called` | Missing initialization | Call `initRecharge()` before any method |
| `Invalid Recharge storefront access token` | Wrong token | Use token starting with `strfnt_` |
| `Customer does not exist in Recharge` | New customer | Handle gracefully, show "add to cart" instead |
| 401 on API call | Session expired | Re-authenticate with login function |
| CDN data stale | Cached | Call `resetCDNCache()` |
| Bundle methods fail | Wrong context | Bundle methods only work in Shopify storefront domain |

---

## Rate Limiting

**Symptom:** `429 Too Many Requests`

Implement exponential backoff with `Retry-After` header:

```javascript
async function retryWithBackoff(fn, maxRetries = 5) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (err) {
      if (err.status === 429 && i < maxRetries - 1) {
        const retryAfter = parseInt(err.headers?.get?.('Retry-After') || '1', 10);
        const delay = Math.max(retryAfter * 1000, Math.pow(2, i) * 1000);
        await new Promise(r => setTimeout(r, delay));
      } else throw err;
    }
  }
}
```

**Best practices:**
- Batch operations where possible (`async_batches`)
- Cache responses when data doesn't change frequently
- Use webhooks instead of polling for state changes
- Add minimum 200ms delay between sequential API calls

---

## Platform-Specific Notes

| Platform | Notes |
|----------|-------|
| RCS (Recharge on Shopify) | Recharge manages checkout; theme engine portal available |
| SCI (Shopify Checkout Integration) | Shopify handles checkout; Affinity portal; selling plans synced |
| Custom/Headless | Full API-first; use Storefront SDK (`@rechargeapps/storefront-client`); CDN for product data |
| BigCommerce | Recharge hosted checkout; limited theme engine support |

---

## Quick Reference: Common Fix Patterns

### "Recharge is broken" / General Reset

```bash
# 1. Check store health
GET /store

# 2. Check webhooks
GET /webhooks
# Fix: Delete stale, re-register needed topics

# 3. Check for failed charges
GET /charges?status=ERROR&limit=50
# Fix: POST /charges/{id}/process

# 4. Check subscription state
GET /subscriptions?customer_id={id}&status=ACTIVE
# Fix: POST /subscriptions/{id}/activate (if cancelled)

# 5. Check payment methods
GET /payment_methods?customer_id={id}
# Fix: Send payment update email
```

### "Subscription not working"

```bash
GET /subscriptions/{id}?include=address,customer,charge_activities
# Check: status, next_charge_scheduled_at, address validity, payment method
```

### "Customer can't manage subscriptions"

```bash
# Theme Engine portal
GET /portal/{hash}/request_objects?token=TOKEN&schema={"customer":{},"subscriptions":[],"settings":{}}

# Or via Admin API
GET /customers?email=customer@email.com
GET /subscriptions?customer_id={id}&include=address
```
