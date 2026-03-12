# Recharge API Reference

Base: `https://api.rechargeapps.com`
Auth: `X-Recharge-Access-Token: <token>`
Pagination: cursor-based, `limit` 50 default / 250 max
Versions: `X-Recharge-Version` header (current: `2021-11`)

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
| POST | `/subscriptions/{id}/gift` | write_subscriptions |

Include options: `address`, `charge_activities`, `customer`, `metafields`, `bundle_selections`

## Charges

| Method | Endpoint | Scope |
|--------|----------|-------|
| GET | `/charges` | read_orders |
| GET | `/charges/{id}` | read_orders |
| POST | `/charges/{id}/process` | write_orders |
| POST | `/charges/{id}/capture` | write_orders |
| POST | `/charges/{id}/refund` | write_orders |
| POST | `/charges/{id}/skip` | write_orders |
| POST | `/charges/{id}/unskip` | write_orders |
| POST | `/charges/{id}/apply_discount` | write_orders |
| POST | `/charges/{id}/remove_discount` | write_orders |
| POST | `/charges/{id}/add_gift` | write_orders |
| POST | `/charges/{id}/remove_gift` | write_orders |

Filter: `status` (queued, skipped, error, success, refunded), `scheduled_at_min/max`

## Customers

| Method | Endpoint | Scope |
|--------|----------|-------|
| POST | `/customers` | write_customers |
| GET | `/customers/{id}` | read_customers |
| PUT | `/customers/{id}` | write_customers |
| DELETE | `/customers/{id}` | write_customers |
| GET | `/customers` | read_customers |
| GET | `/customers/{id}/delivery_schedule` | read_customers |
| GET | `/customers/{id}/credits_summary` | read_customers |

## Addresses

| Method | Endpoint | Scope |
|--------|----------|-------|
| POST | `/addresses` | write_customers |
| GET | `/addresses/{id}` | read_customers |
| PUT | `/addresses/{id}` | write_customers |
| DELETE | `/addresses/{id}` | write_customers |
| GET | `/addresses` | read_customers |
| POST | `/addresses/merge` | write_customers |
| POST | `/addresses/{id}/charges/skip` | write_orders |

Merge: up to 10 source addresses into one target. Currencies must match.

## Payment Methods

| Method | Endpoint | Scope |
|--------|----------|-------|
| POST | `/payment_methods` | write_payment_methods |
| GET | `/payment_methods/{id}` | read_payment_methods |
| PUT | `/payment_methods/{id}` | write_payment_methods |
| DELETE | `/payment_methods/{id}` | write_payment_methods |
| GET | `/payment_methods` | read_payment_methods |

Include: `addresses`

## Orders

| Method | Endpoint | Scope |
|--------|----------|-------|
| GET | `/orders` | read_orders |
| GET | `/orders/{id}` | read_orders |
| PUT | `/orders/{id}` | write_orders |
| DELETE | `/orders/{id}` | write_orders |
| POST | `/orders/clone` | write_orders |
| POST | `/orders/{id}/delay` | write_orders |

## Plans

| Method | Endpoint | Scope |
|--------|----------|-------|
| POST | `/plans` | write_plans |
| GET | `/plans` | read_plans |
| PUT | `/plans` | write_plans |
| DELETE | `/plans` | write_plans |

Bulk create/update/delete supported.

## Webhooks

| Method | Endpoint |
|--------|----------|
| POST | `/webhooks` |
| GET | `/webhooks/{id}` |
| PUT | `/webhooks/{id}` |
| DELETE | `/webhooks/{id}` |
| GET | `/webhooks` |
| POST | `/webhooks/{id}/test` |

See `references/webhooks.md` for topics and payloads.

## Store

| Method | Endpoint |
|--------|----------|
| GET | `/store` |

## Async Batches (Bulk Operations)

```http
POST /async_batches          # Create batch
POST /async_batches/{id}/process  # Start processing
GET  /async_batches/{id}     # Check status
```

## HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 401 | Unauthorized (invalid token) |
| 422 | Unprocessable Entity (validation error) |
| 429 | Rate Limited |
| 500 | Server Error |
| 503 | Service Unavailable |
