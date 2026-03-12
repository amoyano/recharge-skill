# Recharge Webhooks Reference

## Management Endpoints

```http
POST   /webhooks           # Create
GET    /webhooks/{id}      # Retrieve
PUT    /webhooks/{id}      # Update
DELETE /webhooks/{id}      # Delete
GET    /webhooks           # List all
POST   /webhooks/{id}/test # Fire a test event
```

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
- `address/created`
- `address/updated`

### Charge
- `charge/created`
- `charge/failed`
- `charge/max_retries_reached`
- `charge/paid`
- `charge/refunded`
- `charge/uncaptured`
- `charge/upcoming`
- `charge/updated`
- `charge/deleted`

### Customer
- `customer/activated`
- `customer/created`
- `customer/deactivated`
- `customer/payment_method_updated`
- `customer/updated`
- `customer/deleted`

### Order
- `order/cancelled`
- `order/created`
- `order/deleted`
- `order/processed`
- `order/payment_captured`
- `order/upcoming`
- `order/updated`
- `order/success`

### Subscription
- `subscription/activated`
- `subscription/cancelled`
- `subscription/created`
- `subscription/deleted`
- `subscription/skipped`
- `subscription/updated`
- `subscription/unskipped`
- `subscription/paused`

## Enriching Payloads

Use `included_objects` to embed related data:

```json
{
  "topic": "charge/paid",
  "included_objects": ["customer", "payment_methods"]
}
```

`charge_activities` (90-day history) also available as included object.

## Reinitializing Webhooks (Full Reset)

```bash
# 1. List all webhooks
GET /webhooks

# 2. Delete each one
DELETE /webhooks/{id}

# 3. Re-register required topics
POST /webhooks  # one per topic

# 4. Test each
POST /webhooks/{id}/test
```

## Rate Limits

`429` response when threshold exceeded. Implement exponential backoff.

## Troubleshooting

| Issue | Action |
|-------|--------|
| Webhooks not firing | Run `/webhooks/{id}/test`, check endpoint URL |
| Duplicate events | Check for duplicate webhook registrations |
| Missing events | Verify topic list covers needed events |
| Payload missing data | Add `included_objects` to webhook config |
