# Recharge Webhooks Reference

Complete reference for Recharge webhooks — real-time event notifications for subscription lifecycle events. Use this skill when setting up webhook listeners, debugging webhook delivery, enriching payloads with related data, or building event-driven Recharge integrations.

**Docs:** https://developer.rechargepayments.com/2021-11 (Webhooks section)

## Management Endpoints

| Method | Endpoint | Scope |
|--------|----------|-------|
| POST | `/webhooks` | write_webhooks |
| GET | `/webhooks/{id}` | read_webhooks |
| PUT | `/webhooks/{id}` | write_webhooks |
| DELETE | `/webhooks/{id}` | write_webhooks |
| GET | `/webhooks` | read_webhooks |
| POST | `/webhooks/{id}/test` | write_webhooks |

## Create Webhook

```json
POST /webhooks
{
  "address": "https://your-store.com/webhooks/recharge",
  "topic": "subscription/created",
  "included_objects": ["customer", "payment_methods"]
}
```

## All Webhook Topics

### Address
| Topic | Fired When |
|-------|------------|
| `address/created` | New address created |
| `address/updated` | Address modified |

### Charge
| Topic | Fired When |
|-------|------------|
| `charge/created` | New charge created |
| `charge/failed` | Charge payment failed |
| `charge/max_retries_reached` | All retry attempts exhausted |
| `charge/paid` | Charge successfully paid |
| `charge/refunded` | Charge refunded |
| `charge/uncaptured` | Charge authorized but not captured |
| `charge/upcoming` | Charge is upcoming (advance notice) |
| `charge/updated` | Charge modified |
| `charge/deleted` | Charge deleted |

### Customer
| Topic | Fired When |
|-------|------------|
| `customer/activated` | Customer account activated |
| `customer/created` | New customer created |
| `customer/deactivated` | Customer deactivated |
| `customer/payment_method_updated` | Payment method changed |
| `customer/updated` | Customer record modified |
| `customer/deleted` | Customer deleted |

### Order
| Topic | Fired When |
|-------|------------|
| `order/cancelled` | Order cancelled |
| `order/created` | New order created |
| `order/deleted` | Order deleted |
| `order/processed` | Order processed and submitted to platform |
| `order/payment_captured` | Payment captured on order |
| `order/upcoming` | Order is upcoming (advance notice) |
| `order/updated` | Order modified |
| `order/success` | Order successfully completed |

### Subscription
| Topic | Fired When |
|-------|------------|
| `subscription/activated` | Subscription reactivated |
| `subscription/cancelled` | Subscription cancelled |
| `subscription/created` | New subscription created |
| `subscription/deleted` | Subscription deleted |
| `subscription/skipped` | Delivery skipped |
| `subscription/updated` | Subscription modified |
| `subscription/unskipped` | Skip reversed |
| `subscription/paused` | Subscription paused |

---

## Enriching Payloads with `included_objects`

Add related data to webhook payloads to reduce follow-up API calls.

```json
{
  "address": "https://your-endpoint.com/recharge",
  "topic": "charge/paid",
  "included_objects": ["customer", "payment_methods"]
}
```

### Available `included_objects` by Topic

| Resource Topics | Supported `included_objects` |
|----------------|----------------------------|
| `address/*` | `customer`, `payment_methods` |
| `charge/*` | `customer`, `metafields`, `payment_methods` |
| `customer/*` | `addresses`, `metafields`, `payment_methods` |
| `order/*` | `customer`, `metafields` |
| `subscription/*` | `customer`, `metafields` |

`charge_activities` (90-day history) also available as an included object on charges (beta).

---

## Reinitializing Webhooks (Full Reset)

Use when webhooks are stale, pointing to old URLs, or not firing.

```bash
# 1. List all current webhooks
curl -H 'X-Recharge-Access-Token: TOKEN' \
     -H 'X-Recharge-Version: 2021-11' \
     https://api.rechargeapps.com/webhooks

# 2. Delete each webhook
curl -X DELETE -H 'X-Recharge-Access-Token: TOKEN' \
     -H 'X-Recharge-Version: 2021-11' \
     https://api.rechargeapps.com/webhooks/{id}

# 3. Re-register required topics
curl -X POST -H 'X-Recharge-Access-Token: TOKEN' \
     -H 'X-Recharge-Version: 2021-11' \
     -H 'Content-Type: application/json' \
     https://api.rechargeapps.com/webhooks \
     -d '{"address":"https://your-endpoint.com/recharge","topic":"subscription/created","included_objects":["customer"]}'

# 4. Test each webhook
curl -X POST -H 'X-Recharge-Access-Token: TOKEN' \
     -H 'X-Recharge-Version: 2021-11' \
     https://api.rechargeapps.com/webhooks/{id}/test
```

### Programmatic Reset (Node.js)

```javascript
const BASE = 'https://api.rechargeapps.com';
const headers = {
  'X-Recharge-Access-Token': process.env.RECHARGE_TOKEN,
  'X-Recharge-Version': '2021-11',
  'Content-Type': 'application/json'
};

async function resetWebhooks(endpointUrl, topics) {
  // Delete existing
  const { webhooks } = await fetch(`${BASE}/webhooks`, { headers }).then(r => r.json());
  for (const wh of webhooks) {
    await fetch(`${BASE}/webhooks/${wh.id}`, { method: 'DELETE', headers });
  }

  // Re-register
  for (const topic of topics) {
    const res = await fetch(`${BASE}/webhooks`, {
      method: 'POST',
      headers,
      body: JSON.stringify({
        address: endpointUrl,
        topic,
        included_objects: ['customer', 'metafields']
      })
    });
    const { webhook } = await res.json();
    
    // Test
    await fetch(`${BASE}/webhooks/${webhook.id}/test`, {
      method: 'POST',
      headers
    });
  }
}

const criticalTopics = [
  'subscription/created', 'subscription/updated', 'subscription/cancelled',
  'charge/paid', 'charge/failed', 'charge/max_retries_reached',
  'customer/created', 'customer/updated',
  'order/created', 'order/processed'
];

resetWebhooks('https://your-endpoint.com/recharge', criticalTopics);
```

---

## Webhook Payload Structure

All webhook payloads follow this pattern:

```json
{
  "subscription": {
    "id": 123,
    "address_id": 456,
    "customer_id": 789,
    "status": "ACTIVE",
    ...
  }
}
```

With `included_objects`, additional top-level keys are added:

```json
{
  "subscription": { ... },
  "customer": { ... },
  "metafields": [ ... ]
}
```

---

## Best Practices

1. **Always use `included_objects`** to get related data and reduce API calls
2. **Implement idempotency** — webhooks may fire more than once for the same event
3. **Return 2xx quickly** — process async, don't block the webhook response
4. **Handle `charge/max_retries_reached`** — this is critical for dunning management
5. **Monitor for `charge/failed`** — set up alerts for payment failures
6. **Deduplicate** by checking for duplicate webhook registrations (`GET /webhooks`)
7. **Use `POST /webhooks/{id}/test`** after registration to verify delivery

---

## Troubleshooting

| Issue | Action |
|-------|--------|
| Webhooks not firing | Run `POST /webhooks/{id}/test`, check endpoint URL is reachable |
| Duplicate events | Check for duplicate webhook registrations via `GET /webhooks` |
| Missing events | Verify topic list covers needed events |
| Payload missing data | Add `included_objects` to webhook config |
| Stale webhook URLs | Delete and re-register with correct endpoint |
| 429 rate limit on webhook processing | Implement queue with exponential backoff |
| Webhook test succeeds but real events fail | Check for IP allowlisting, SSL cert issues, or timeout on endpoint |
