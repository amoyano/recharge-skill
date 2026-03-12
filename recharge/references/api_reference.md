# Recharge Admin API Reference (v2021-11)

Complete reference for the Recharge REST API. Use this skill when building server-side integrations, managing subscriptions programmatically, processing charges, syncing data, or troubleshooting Recharge API issues.

**Docs:** https://developer.rechargepayments.com/2021-11

## Fundamentals

| Setting | Value |
|---------|-------|
| Base URL | `https://api.rechargeapps.com` |
| Auth header | `X-Recharge-Access-Token: <token>` |
| Version header | `X-Recharge-Version: 2021-11` |
| Content-Type | `application/json` |
| Pagination | Cursor-based (`next_cursor` / `previous_cursor`) |
| Default limit | 50 (max 250) |

## Platform Tags

| Tag | Checkout | Ecommerce Platform |
|-----|----------|-------------------|
| BigCommerce | Recharge hosted | BigCommerce |
| Custom | Recharge hosted or API-first | Custom |
| RCS | Recharge hosted | Shopify |
| SCI | Shopify hosted | Shopify |

## API Token Scopes

| Write | Read |
|-------|------|
| | `read_accounts` |
| `write_batches` | `read_batches` |
| `write_customers` | `read_customers` |
| `write_discounts` | `read_discounts` |
| | `read_events` |
| `write_notifications` | |
| `write_orders` | `read_orders` |
| `write_payment_methods` | `read_payment_methods` |
| `write_plans` | `read_plans` |
| `write_products` | `read_products` |
| `write_subscriptions` | `read_subscriptions` |
| | `read_store` |
| | `read_credit_accounts` |
| | `read_credit_adjustments` |

## HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | OK |
| 201 | Created (resource in body) |
| 202 | Accepted (processing) |
| 204 | No Content (deleted) |
| 400 | Bad Request (missing required param) |
| 401 | Unauthorized (invalid API key) |
| 402 | Request Failed (valid params, failed request) |
| 403 | Forbidden (permission scope error or uninstalled store) |
| 404 | Not Found |
| 405 | Method Not Allowed |
| 406 | Not Acceptable |
| 409 | Conflict (concurrent edit on address/children) |
| 415 | Not JSON body |
| 422 | Unprocessable Entity (validation error) |
| 426 | Invalid API version |
| 429 | Rate Limited |
| 500 | Internal Server Error |
| 501 | Not Implemented (may be in future version) |
| 503 | 3rd party service timeout |

## Cursor Pagination

```bash
# First page
GET /subscriptions?limit=250

# Next page (use next_cursor from response)
GET /subscriptions?limit=250&cursor=<next_cursor_value>

# Previous page
GET /subscriptions?limit=250&cursor=<previous_cursor_value>
```

No total count available in v2021-11. Use `next_cursor`/`previous_cursor` for pagination UI.

## Extending Responses (Includes)

Use `include` query param on GET requests or `included_objects` on webhooks.

| Resource | Supported `include` values | Webhook `included_objects` |
|----------|---------------------------|---------------------------|
| Addresses | `charge_activities`, `customer`, `discount`, `payment_methods`, `subscriptions` | `customer`, `payment_methods` |
| Charges | `charge_activities` (beta), `customer`, `metafields`, `payment_methods` | `customer`, `metafields`, `payment_methods` |
| Customers | `addresses`, `metafields`, `payment_methods`, `subscriptions` | `addresses`, `metafields`, `payment_methods` |
| Orders | `customer`, `metafields`, `subscriptions` | `customer`, `metafields` |
| Payment Methods | `addresses` | `addresses` |
| Subscriptions | `address`, `charge_activities`, `customer`, `metafields`, `bundle_product`, `bundle_selections` | `customer`, `metafields` |

```bash
# Example: include customer and metafields
GET /subscriptions/123?include=customer,metafields
```

## Sorting

Use `sort_by` query param. Format: `field-direction`.

| Resource | Available sort_by |
|----------|------------------|
| Address | `id-asc`, `id-desc`, `updated_at-asc`, `updated_at-desc` |
| Charge | `id-asc`, `id-desc`, `created_at-asc/desc`, `updated_at-asc/desc`, `scheduled_at-asc/desc` |
| Customer | `id-asc`, `id-desc`, `created_at-asc/desc`, `updated_at-asc/desc` |
| Discount | `id-asc`, `id-desc`, `created_at-asc/desc`, `updated_at-asc/desc` |
| Order | `id-asc`, `id-desc`, `updated_at-asc/desc`, `processed_at-asc/desc`, `scheduled_at-asc/desc` |
| Subscription | `id-asc`, `id-desc`, `created_at-asc/desc`, `updated_at-asc/desc` |

---

## Addresses

Shipping address records. Each customer can have multiple. Subscriptions are children of addresses.

| Method | Endpoint | Scope |
|--------|----------|-------|
| POST | `/addresses` | write_customers |
| GET | `/addresses/{id}` | read_customers |
| PUT | `/addresses/{id}` | write_customers |
| DELETE | `/addresses/{id}` | write_customers |
| GET | `/addresses` | read_customers |
| POST | `/addresses/merge` | write_customers |
| POST | `/addresses/{id}/charges/skip` | write_orders |

**Key fields:** `id`, `customer_id`, `payment_method_id`, `address1`, `address2`, `city`, `company`, `country_code`, `first_name`, `last_name`, `phone`, `province`, `zip`, `presentment_currency`, `discounts[]`, `order_attributes[]`, `order_note`, `shipping_lines_override[]`

**Create required:** `customer_id`, `address1`, `city`, `country_code`, `first_name`, `last_name`, `phone`, `province`, `zip`

**List filters:** `customer_id`, `discount_id`, `discount_code`, `ids`, `is_active`, `created_at_min/max`, `updated_at_min/max`

**Merge:** Up to 10 source addresses into 1 target. Currencies must match.
```json
POST /addresses/merge
{
  "target_address": {"id": 123},
  "source_addresses": [{"id": 456}, {"id": 789}],
  "delete_source_addresses": true
}
```

**Skip future charge:**
```json
POST /addresses/{id}/charges/skip
{
  "date": "2024-09-15",
  "subscription_ids": [27363808, 27363809]
}
```

---

## Bundle Selections

Contents within a Bundle linked to a Subscription. Recharge Pro only.

| Method | Endpoint | Scope |
|--------|----------|-------|
| GET | `/bundle_selections` | read_subscriptions |
| GET | `/bundle_selections/{id}` | read_subscriptions |
| POST | `/bundle_selections` | write_subscriptions |
| PUT | `/bundle_selections/{id}` | write_subscriptions |
| DELETE | `/bundle_selections/{id}` | write_subscriptions |

**Key fields:** `id`, `bundle_variant_id`, `purchase_item_id`, `external_product_id`, `external_variant_id`, `items[]`

**Items array:** `id`, `collection_id`, `collection_source`, `external_product_id`, `external_variant_id`, `quantity`

**List filters:** `purchase_item_ids`, `bundle_variant_ids`, `active_purchase_items`

---

## Charges

Financial transactions for subscriptions (past or future).

| Method | Endpoint | Scope |
|--------|----------|-------|
| GET | `/charges/{id}` | read_orders |
| GET | `/charges` | read_orders |
| POST | `/charges/{id}/apply_discount` | write_orders |
| POST | `/charges/{id}/remove_discount` | write_orders |
| POST | `/charges/{id}/skip` | write_orders |
| POST | `/charges/{id}/unskip` | write_orders |
| POST | `/charges/{id}/refund` | write_orders |
| POST | `/charges/{id}/process` | write_orders |
| POST | `/charges/{id}/capture_payment` | write_orders |

**Charge statuses:** `QUEUED`, `SKIPPED`, `ERROR`, `SUCCESS`, `REFUNDED`, `PARTIALLY_REFUNDED`, `UNCAPTURED`

**Key fields:** `id`, `address_id`, `customer`, `status`, `scheduled_at`, `processed_at`, `currency`, `subtotal_price`, `total_price`, `total_tax`, `total_discounts`, `total_refunds`, `line_items[]`, `shipping_address`, `billing_address`, `shipping_lines[]`, `tax_lines[]`, `discounts[]`, `error`, `error_type`, `retry_date`, `payment_processor`, `external_order_id`, `external_transaction_id`, `has_uncommitted_changes`, `tags`, `type`

**List filters:** `status`, `address_id`, `customer_id`, `ids`, `scheduled_at_min/max`, `created_at_min/max`, `updated_at_min/max`, `purchase_item_id`

```bash
# Find failed charges
GET /charges?status=ERROR&limit=250

# Retry a failed charge
POST /charges/{id}/process

# Skip a charge (provide purchase_item_ids)
POST /charges/{id}/skip
{"purchase_item_ids": [123, 456]}

# Apply discount
POST /charges/{id}/apply_discount
{"discount_code": "SAVE10"}
```

---

## Customers

| Method | Endpoint | Scope |
|--------|----------|-------|
| POST | `/customers` | write_customers |
| GET | `/customers/{id}` | read_customers |
| PUT | `/customers/{id}` | write_customers |
| DELETE | `/customers/{id}` | write_customers |
| GET | `/customers` | read_customers |
| GET | `/customers/{id}/delivery_schedule` | read_customers |

**Key fields:** `id`, `email`, `first_name`, `last_name`, `phone`, `external_customer_id`, `hash`, `status`, `first_charge_processed_at`, `number_active_subscriptions`, `number_subscriptions`, `has_payment_method_in_dunning`, `tax_exempt`

**List filters:** `email`, `ids`, `external_customer_id`, `created_at_min/max`, `updated_at_min/max`

**Delivery schedule:** Returns upcoming delivery dates for all subscriptions.

---

## Discounts

| Method | Endpoint | Scope |
|--------|----------|-------|
| POST | `/discounts` | write_discounts |
| GET | `/discounts/{id}` | read_discounts |
| PUT | `/discounts/{id}` | write_discounts |
| DELETE | `/discounts/{id}` | write_discounts |
| GET | `/discounts` | read_discounts |

**Key fields:** `id`, `code`, `value`, `discount_type` (percentage/fixed_amount), `duration` (forever/usage_limit/single_use), `status`, `applies_to_product_type`, `starts_at`, `ends_at`, `usage_limit`, `times_used`, `once_per_customer`, `channel_settings`

---

## Metafields

| Method | Endpoint | Scope |
|--------|----------|-------|
| POST | `/metafields` | write on owner resource |
| GET | `/metafields/{id}` | read on owner resource |
| PUT | `/metafields/{id}` | write on owner resource |
| DELETE | `/metafields/{id}` | write on owner resource |
| GET | `/metafields` | read on owner resource |

**Owner types:** `store`, `customer`, `subscription`, `order`, `charge`, `address`

---

## Onetimes (One-Time Products)

| Method | Endpoint | Scope |
|--------|----------|-------|
| POST | `/onetimes` | write_subscriptions |
| GET | `/onetimes/{id}` | read_subscriptions |
| PUT | `/onetimes/{id}` | write_subscriptions |
| DELETE | `/onetimes/{id}` | write_subscriptions |
| GET | `/onetimes` | read_subscriptions |

---

## Orders

| Method | Endpoint | Scope |
|--------|----------|-------|
| GET | `/orders/{id}` | read_orders |
| GET | `/orders` | read_orders |
| PUT | `/orders/{id}` | write_orders |
| DELETE | `/orders/{id}` | write_orders |
| POST | `/orders/{id}/clone` | write_orders |
| POST | `/orders/{id}/delay` | write_orders |

**Key fields:** `id`, `charge_id`, `address_id`, `customer`, `status`, `scheduled_at`, `processed_at`, `line_items[]`, `shipping_address`, `shipping_lines[]`, `total_price`, `subtotal_price`, `external_order_id`, `external_order_number`, `tags`, `type`, `is_prepaid`

**List filters:** `status`, `address_id`, `charge_id`, `customer_id`, `ids`, `scheduled_at_min/max`, `created_at_min/max`, `updated_at_min/max`, `purchase_item_id`, `external_order_id`

---

## Payment Methods

| Method | Endpoint | Scope |
|--------|----------|-------|
| POST | `/payment_methods` | write_payment_methods |
| GET | `/payment_methods/{id}` | read_payment_methods |
| PUT | `/payment_methods/{id}` | write_payment_methods |
| DELETE | `/payment_methods/{id}` | write_payment_methods |
| GET | `/payment_methods` | read_payment_methods |

**Key fields:** `id`, `customer_id`, `payment_type` (CREDIT_CARD, PAYPAL, APPLE_PAY, GOOGLE_PAY, SEPA_DEBIT), `status`, `payment_details` (brand, last4, exp_month, exp_year), `default`, `billing_address`, `processor_name`

**List filters:** `customer_id`

---

## Plans

Selling plans for subscription products.

| Method | Endpoint | Scope |
|--------|----------|-------|
| POST | `/plans` | write_plans |
| GET | `/plans` | read_plans |
| PUT | `/plans` | write_plans |
| DELETE | `/plans` | write_plans |

Bulk create/update/delete supported.

**Key fields:** `id`, `title`, `external_product_id`, `discount_amount`, `discount_type`, `order_interval_frequency`, `order_interval_unit` (day/week/month), `charge_interval_frequency`, `type` (subscription/prepaid/onetime), `sort_order`

---

## Products

| Method | Endpoint | Scope |
|--------|----------|-------|
| GET | `/products` | read_products |
| GET | `/products/{id}` | read_products |

---

## Subscriptions

| Method | Endpoint | Scope |
|--------|----------|-------|
| POST | `/subscriptions` | write_subscriptions |
| GET | `/subscriptions/{id}` | read_subscriptions |
| PUT | `/subscriptions/{id}` | write_subscriptions |
| DELETE | `/subscriptions/{id}` | write_subscriptions |
| GET | `/subscriptions` | read_subscriptions |
| POST | `/subscriptions/{id}/set_next_charge_date` | write_subscriptions |
| POST | `/subscriptions/{id}/change_address` | write_subscriptions |
| POST | `/subscriptions/{id}/cancel` | write_subscriptions |
| POST | `/subscriptions/{id}/activate` | write_subscriptions |

**Subscription statuses:** `ACTIVE`, `CANCELLED`, `EXPIRED`

**Key fields:** `id`, `address_id`, `customer_id`, `status`, `next_charge_scheduled_at`, `cancelled_at`, `cancellation_reason`, `price`, `quantity`, `sku`, `product_title`, `variant_title`, `external_product_id`, `external_variant_id`, `order_interval_frequency`, `order_interval_unit`, `charge_interval_frequency`, `expire_after_specific_number_of_charges`, `order_day_of_month`, `order_day_of_week`, `properties[]`, `max_retries_reached`

**Create required:** `address_id`, `next_charge_scheduled_at`, `quantity`, `external_variant_id`, `external_product_id`, plus one of: (`charge_interval_frequency` + `order_interval_frequency` + `order_interval_unit`) or `plan_id`

**List filters:** `address_id`, `customer_id`, `status`, `ids`, `external_variant_id`, `created_at_min/max`, `updated_at_min/max`

Include options: `address`, `charge_activities`, `customer`, `metafields`, `bundle_product`, `bundle_selections`

```bash
# Create subscription
POST /subscriptions
{
  "address_id": 123,
  "next_charge_scheduled_at": "2024-02-01",
  "quantity": 1,
  "external_variant_id": {"ecommerce": "12345"},
  "external_product_id": {"ecommerce": "67890"},
  "charge_interval_frequency": 30,
  "order_interval_frequency": 30,
  "order_interval_unit": "day"
}

# Cancel
POST /subscriptions/{id}/cancel
{"cancellation_reason": "Too expensive"}

# Reactivate
POST /subscriptions/{id}/activate

# Set next charge date
POST /subscriptions/{id}/set_next_charge_date
{"date": "2024-03-01"}

# Change address
POST /subscriptions/{id}/change_address
{"address_id": 456}
```

---

## Store

| Method | Endpoint | Scope |
|--------|----------|-------|
| GET | `/store` | read_store |

Returns store configuration, currency, payment processor, timezone, and feature flags.

---

## Webhooks

| Method | Endpoint | Scope |
|--------|----------|-------|
| POST | `/webhooks` | write_webhooks |
| GET | `/webhooks/{id}` | read_webhooks |
| PUT | `/webhooks/{id}` | write_webhooks |
| DELETE | `/webhooks/{id}` | write_webhooks |
| GET | `/webhooks` | read_webhooks |
| POST | `/webhooks/{id}/test` | write_webhooks |

See `references/webhooks.md` for all topics and payloads.

---

## Async Batches (Bulk Operations)

```bash
POST /async_batches          # Create batch
POST /async_batches/{id}/process  # Start processing
GET  /async_batches/{id}     # Check status
```

**Batch types:** `subscription_update`, `address_update`, `subscription_cancel`

---

## Events

| Method | Endpoint | Scope |
|--------|----------|-------|
| GET | `/events` | read_events |
| GET | `/events/{id}` | read_events |

Read-only audit log of actions in the Recharge system.

---

## Accounts

| Method | Endpoint | Scope |
|--------|----------|-------|
| GET | `/accounts` | read_accounts |

Returns info about the API token's associated account.

---

## Notifications

| Method | Endpoint | Scope |
|--------|----------|-------|
| POST | `/notifications/send_email` | write_notifications |

Send transactional emails to customers (payment update, upcoming charge, etc.).

---

## Rate Limiting

429 responses include `Retry-After` header. Implement exponential backoff:

```javascript
async function rechargeRequest(url, options, maxRetries = 5) {
  for (let i = 0; i < maxRetries; i++) {
    const res = await fetch(url, options);
    if (res.status === 429 && i < maxRetries - 1) {
      const retryAfter = parseInt(res.headers.get('Retry-After') || '1', 10);
      await new Promise(r => setTimeout(r, retryAfter * 1000));
      continue;
    }
    if (!res.ok) throw new Error(`Recharge API ${res.status}: ${await res.text()}`);
    if (res.status === 204) return null;
    return res.json();
  }
}
```
