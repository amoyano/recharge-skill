# Recharge Troubleshooting & Common Fixes

## Diagnostic Decision Tree

```
Recharge issue reported
├── Webhooks not firing?         → See "Webhook Issues"
├── Charges failing/erroring?    → See "Charge Failures"
├── Subscriptions out of sync?   → See "Subscription Sync"
├── Payment method issues?       → See "Payment Problems"
├── API auth errors?             → See "Authentication Errors"
└── Orders not generating?       → See "Order Generation Issues"
```

## Webhook Issues

**Symptom:** Events not received, webhooks silent

```http
# Step 1: List all registered webhooks
GET /webhooks

# Step 2: Test a specific webhook
POST /webhooks/{id}/test

# Step 3: If stale/broken, delete and re-register
DELETE /webhooks/{id}
POST /webhooks
{
  "address": "https://your-endpoint.com/recharge",
  "topic": "subscription/updated"
}
```

**Common causes:**
- Endpoint URL changed (store migration, new domain)
- Webhook registered to old staging URL
- Duplicate webhooks causing conflicts
- Missing `included_objects` for expected payload fields

## Charge Failures

**Symptom:** `charge/failed` or `charge/max_retries_reached` events

```http
# Find all failed charges
GET /charges?status=error&limit=250

# Retry a specific charge
POST /charges/{id}/process

# Check charge details and activities
GET /charges/{id}?include=charge_activities,customer,payment_methods
```

**Charge statuses:**
| Status | Meaning |
|--------|---------|
| `queued` | Scheduled, not yet processed |
| `skipped` | Manually or automatically skipped |
| `error` | Failed, can be retried |
| `success` | Processed successfully |
| `refunded` | Refunded after success |
| `uncaptured` | Authorized but not captured |

## Subscription Sync

**Symptom:** Subscription state stale or out of sync with Shopify

```http
# Force a sync by reading + updating with same data
GET /subscriptions/{id}
PUT /subscriptions/{id} { "status": "ACTIVE" }

# Reactivate a cancelled subscription
POST /subscriptions/{id}/activate

# Check full subscription with all includes
GET /subscriptions/{id}?include=address,customer,metafields,charge_activities
```

## Payment Problems

**Symptom:** Payment method invalid, expired, or missing

```http
# Get customer's payment methods
GET /payment_methods?customer_id={id}

# Update payment method
PUT /payment_methods/{id}
{
  "payment_token": "new_token",
  "payment_type": "CREDIT_CARD"
}

# Send update email to customer
POST /customers/{id}/send_payment_update_email
```

## Authentication Errors

**Symptom:** `401 Unauthorized`

- Verify token is correct: `GET /store` should return store info
- Check required scopes for the operation
- Token must be passed as header: `X-Recharge-Access-Token: <token>`
- Different tokens for different stores — confirm correct store token

**Common scopes needed:**
| Operation | Scope |
|-----------|-------|
| Read subscriptions | `read_subscriptions` |
| Modify subscriptions | `write_subscriptions` |
| Process charges | `write_orders` |
| Customer data | `read_customers` / `write_customers` |
| Webhooks | `read_webhooks` / `write_webhooks` |

## Order Generation Issues

**Symptom:** Orders not being created from subscriptions

```http
# Check upcoming orders
GET /orders?status=QUEUED&limit=50

# Check customer delivery schedule
GET /customers/{id}/delivery_schedule

# Verify subscription next charge date
GET /subscriptions/{id}

# Manually force next charge date
POST /subscriptions/{id}/set_next_charge_date
{ "date": "2024-01-15" }
```

## Address / Merge Issues

**Symptom:** Subscriptions split across multiple addresses, duplicate data

```http
# Merge addresses (max 10 sources)
POST /addresses/merge
{
  "next_address_id": 123,        # Target address
  "previous_address_ids": [456]  # Sources to merge in
}
```

Note: Cannot merge if addresses have different `presentment_currency`.

## Theme Portal Issues (Customer-Facing)

For issues in the Recharge customer portal (theme layer):

```javascript
// Re-fetch customer state
const schema = '{ "customer": {}, "subscriptions": [], "payment_methods": [] }';
const url = `/portal/${customerHash}/request_objects?token=${token}&schema=${schema}`;
const data = await fetch(url).then(r => r.json());
```

## Rate Limiting

`429 Too Many Requests` — implement exponential backoff:

```javascript
async function retryWithBackoff(fn, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (err) {
      if (err.status === 429 && i < maxRetries - 1) {
        await new Promise(r => setTimeout(r, Math.pow(2, i) * 1000));
      } else throw err;
    }
  }
}
```

## Platform-Specific Notes

| Platform | Notes |
|----------|-------|
| RCS (Recharge on Shopify) | Recharge manages checkout |
| SCI (Shopify checkout) | Shopify handles checkout flow |
| Custom/Headless | Full API-first, use Storefront SDK |
