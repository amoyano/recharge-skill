# Recharge Theme Engine Reference

Complete reference for the Recharge Theme Engine — the customer portal customization layer embedded in Shopify themes. Use this skill when building or customizing the Recharge customer portal, working with Jinja templates, creating subscription management UIs, or handling portal API calls.

**Docs:** https://theme.rechargepayments.com/

## Overview

The Theme Engine powers the Recharge Customer Portal within Shopify themes. It uses **Jinja templating** (similar to Shopify Liquid) with access to Recharge data objects, filters, and routes.

| Layer | Purpose |
|-------|---------|
| Theme Engine Templates | Jinja-based HTML templates for portal pages |
| Theme Engine Routes | URL endpoints that render templates and accept data |
| Theme Engine API | JSON endpoints for reading/writing data via JS |
| Request Objects | Schema-based JSON fetching system |

## Base Paths

| Context | Base Path |
|---------|-----------|
| Embedded (Shopify proxy) | `/tools/recurring/portal/{customer_hash}/` |
| Hosted portal | `/portal/{customer_hash}/` |
| Shopify proxy URL (Jinja) | `{{ shopify_proxy_url if proxy_redirect else "" }}` |

## Token Security

All portal requests require a customer token appended as `?token=<value>`. Tokens are:
- Auto-generated when customer signs in
- Expiring (time-limited session)
- Verified against the customer account
- Available as `window.customerToken` in portal JS

If invalid/missing token, redirects to login page.

---

## Request Objects (Schema-Based Data Fetching)

The primary way to fetch data from the Theme Engine API. Pass a JSON schema to get exactly the objects you need in a single call.

### Endpoint

```
GET /portal/{hash}/request_objects?token={token}&schema={json_schema}
```

### Available Schemas

| Object | Schema | Description |
|--------|--------|-------------|
| Customer | `{ "customer": {} }` | Current customer data |
| Payment Sources | `{ "payment_sources": [] }` | All payment sources (legacy) |
| Payment Methods | `{ "payment_methods": [] }` | All payment methods (Novum 4+) |
| Specific Payment Method | `{ "payment_methods": [{ "id": 123 }] }` | By ID |
| Addresses | `{ "addresses": [] }` | All addresses |
| Specific Address | `{ "address": { "id": 123 } }` | By ID |
| Subscriptions | `{ "subscriptions": [] }` | All subscriptions |
| Specific Subscription | `{ "subscription": { "id": 123 } }` | By ID |
| Bundle Selections | `{ "subscription": { "id": 123, "bundle_selections": {} } }` | With bundles |
| Orders | `{ "orders": [] }` | All orders |
| Specific Order | `{ "order": { "id": 123 } }` | By ID |
| Memberships | `{ "memberships": [] }` | All memberships |
| Specific Membership | `{ "membership": { "id": 123 } }` | By ID |
| Memberships w/ Program | `{ "memberships": { "include": "membership_program" } }` | With program info |
| Delivery Schedule | `{ "delivery_schedule": [] }` | Upcoming deliveries |
| Settings | `{ "settings": {} }` | Portal settings |
| Store | `{ "store": {} }` | Store configuration |
| Products | `{ "products": [] }` | Product search |
| Shipping Countries | `{ "shipping_countries": [] }` | Available shipping countries |
| Retention Strategies | `{ "retention_strategies": [] }` | Cancellation retention flows |

### Combining Multiple Objects

```javascript
const schema = JSON.stringify({
  customer: {},
  subscriptions: [],
  addresses: [],
  payment_methods: [],
  orders: [],
  settings: {},
  store: {}
});
const url = `/portal/${hash}/request_objects?token=${token}&schema=${schema}`;
const data = await fetch(url).then(r => r.json());
```

### Nested Includes

```javascript
// Address with discount details
const schema = JSON.stringify({
  addresses: { discount: { id: "parent.discount_id" } },
  customer: {},
  settings: {},
  store: {}
});
```

---

## Page-Specific Schemas

### Customer Pages
| Page URL Variable | Schema |
|-------------------|--------|
| `show_customer_url` | `{ "customer": {}, "payment_sources": [] }` |
| `update_customer_url` | `{ "customer": {} }` |

### Payment Methods (Novum 4+)
| Page URL Variable | Schema |
|-------------------|--------|
| `payment_methods_list_url` | `{ "customer": {}, "payment_methods": [] }` |
| `show_payment_method_url` | `{ "customer": {}, "payment_methods": [{ "id": ID }] }` |

### Addresses
| Page URL Variable | Schema |
|-------------------|--------|
| `list_addresses_url` | `{ "customer": {}, "addresses": [] }` |
| `show_address_url` | `{ "customer": {}, "address": { "id": ID } }` |
| `create_address_url` | `{ "customer": {} }` |

### Subscriptions
| Page URL Variable | Schema |
|-------------------|--------|
| `list_subscriptions_url` | `{ "customer": {}, "addresses": [], "subscriptions": [] }` |
| `show_subscription_url` | `{ "customer": {}, "subscription": { "id": ID } }` |
| `create_subscription_url` | `{ "customer": {}, "addresses": [] }` |

### Subscription Actions
| Page URL Variable | Schema |
|-------------------|--------|
| `activate_subscription_url` | `{ "customer": {}, "subscription": { "id": ID } }` |
| `skip_subscription_url` | `{ "customer": {}, "subscription": { "id": ID } }` |
| `unskip_subscription_url` | `{ "customer": {}, "subscription": { "id": ID } }` |
| `delay_subscription_url` | `{ "customer": {}, "subscription": { "id": ID } }` |
| `cancel_subscription_url` | `{ "customer": {}, "subscription": { "id": ID } }` |
| `retention_strategy_url` | `{ "customer": {}, "subscription": { "id": ID }, "retention_strategies": [] }` |

### Orders
| Page URL Variable | Schema |
|-------------------|--------|
| `list_orders_url` | `{ "customer": {}, "orders": [] }` |
| `show_order_url` | `{ "customer": {}, "order": { "id": ID } }` |

### Discounts
| Page URL Variable | Schema |
|-------------------|--------|
| `apply_discount_url` | `{ "customer": {}, "address": { "id": ID } }` |
| `remove_discount_from_address_url` | `{ "customer": {}, "address": { "id": ID } }` |

### Memberships
| Page URL Variable | Schema |
|-------------------|--------|
| `membership_list_url` | `{ "customer": {}, "addresses": [], "memberships": [] }` |
| `membership_activate_url` | `{ "customer": {}, "membership": { "id": ID } }` |
| `membership_cancel_url` | `{ "customer": {}, "membership": { "id": ID } }` |
| `membership_retention_strategy_url` | `{ "customer": {}, "membership": { "id": ID }, "retention_strategies": { "sort_by": "id-asc", "cancellation_flow_type": "membership" } }` |

---

## Core Routes (REST Endpoints)

### Addresses

| Method | Route | Description | Template |
|--------|-------|-------------|----------|
| GET | `/portal/{hash}/addresses` | List addresses | `addresses.html` |
| GET | `/portal/{hash}/addresses/new` | New address form | `address_new.html` |
| POST | `/portal/{hash}/addresses` | Create address | - |
| GET | `/portal/{hash}/addresses/{id}` | Show/edit address | `address.html` |
| POST | `/portal/{hash}/addresses/{id}` | Update address | - |
| POST | `/portal/{hash}/addresses/{id}/charges/skip` | Skip charges | - |
| POST | `/portal/{hash}/addresses/merge` | Merge addresses | - |

**Create/Update params:** `first_name`\*, `last_name`\*, `address1`\*, `address2`, `city`\*, `company`, `country`\*, `province`\*, `zip`\*, `phone`

```javascript
// Create address
const data = {
  address1: "101 Washington Street",
  city: "Los Angeles",
  country: "United States",
  first_name: "John",
  last_name: "Doe",
  province: "California",
  zip: "90025",
  phone: "5551231234"
};
await fetch(addressListUrl, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(data)
}).then(r => r.json());
```

### Subscriptions

| Method | Route | Description | Template |
|--------|-------|-------------|----------|
| GET | `/portal/{hash}/subscriptions` | List subscriptions | `subscriptions.html` |
| POST | `/portal/{hash}/subscriptions` | Create subscription | - |
| POST | `/portal/{hash}/subscriptions/bulk` | Bulk create | - |
| GET | `/portal/{hash}/subscriptions/{id}` | Show subscription | `subscription.html` |
| POST | `/portal/{hash}/subscriptions/{id}` | Update subscription | - |
| POST | `/portal/{hash}/subscriptions/{id}/cancel` | Cancel | `subscription_cancel.html` |
| POST | `/portal/{hash}/subscriptions/{id}/activate` | Reactivate | - |
| POST | `/portal/{hash}/subscriptions/{id}/skip` | Skip next delivery | - |
| POST | `/portal/{hash}/subscriptions/{id}/unskip` | Unskip | - |
| POST | `/portal/{hash}/subscriptions/{id}/delay` | Delay delivery | - |
| POST | `/portal/{hash}/subscriptions/{id}/swap` | Swap product | - |
| POST | `/portal/{hash}/subscriptions/{id}/charge_date` | Change charge date | - |

```javascript
// Update subscription frequency
await fetch(`/portal/${hash}/subscriptions/${id}`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    quantity: 2,
    order_interval_frequency: "30",
    order_interval_unit: "day"
  })
}).then(r => r.json());

// Cancel subscription
await fetch(`/portal/${hash}/subscriptions/${id}/cancel`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    cancellation_reason: "Too expensive"
  })
}).then(r => r.json());

// Swap product
await fetch(`/portal/${hash}/subscriptions/${id}/swap`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    shopify_variant_id: "new_variant_id"
  })
}).then(r => r.json());
```

### Charges

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/portal/{hash}/charges` | List charges |
| POST | `/portal/{hash}/charges/process` | Process a charge |
| POST | `/portal/{hash}/charges/skip` | Skip a charge |
| POST | `/portal/{hash}/charges/unskip` | Unskip a charge |

### Payment Methods (Novum 4+)

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/portal/{hash}/payment_methods` | List payment methods |
| GET | `/portal/{hash}/payment_methods/{id}` | Show payment method |
| POST | `/portal/{hash}/payment_methods/{id}` | Update payment method |

### Customer

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/portal/{hash}/customer` | Show customer |
| POST | `/portal/{hash}/customer` | Update customer |

### Orders

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/portal/{hash}/orders` | List orders |
| GET | `/portal/{hash}/orders/{id}` | Show order |
| POST | `/portal/{hash}/orders/{id}/delay` | Delay order |

### One-Time Products

| Method | Route | Description |
|--------|-------|-------------|
| POST | `/portal/{hash}/onetime` | Create one-time |
| GET | `/portal/{hash}/onetime/{id}` | Show one-time |
| POST | `/portal/{hash}/onetime/{id}` | Update one-time |
| POST | `/portal/{hash}/onetime/{id}/cancel` | Cancel one-time |
| POST | `/portal/{hash}/onetime/{id}/charge_date` | Change charge date |

### Discounts

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/portal/{hash}/discounts` | List discounts |
| POST | `/portal/{hash}/discounts/apply` | Apply discount |
| POST | `/portal/{hash}/discounts/remove` | Remove discount |

### Memberships

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/portal/{hash}/memberships` | List memberships |
| GET | `/portal/{hash}/memberships/{id}` | Show membership |
| POST | `/portal/{hash}/memberships/{id}/activate` | Activate |
| POST | `/portal/{hash}/memberships/{id}/cancel` | Cancel |

### Utility Endpoints

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/portal/{hash}/schedule` | Delivery schedule |
| GET | `/portal/{hash}/settings` | Portal settings |
| GET | `/portal/{hash}/shop` | Shop object |
| GET | `/portal/{hash}/store` | Store object |
| GET | `/portal/{hash}/products` | Product search |

---

## Jinja Template Objects

### Address Object

| Property | Type | Description |
|----------|------|-------------|
| `address.id` | number | Unique ID |
| `address.address1` | string | Street address |
| `address.address2` | string | Additional info |
| `address.city` | string | City |
| `address.company` | string | Company |
| `address.country` | string | Country name |
| `address.province` | string | State/province |
| `address.zip` | string | Postal code |
| `address.phone` | string | Phone number |
| `address.first_name` | string | First name |
| `address.last_name` | string | Last name |
| `address.customer_id` | number | Customer ID |
| `address.discount` | object | Applied discount |
| `address.discount_id` | number | Discount ID |
| `address.cart_note` | string | Order note |
| `address.presentment_currency` | string | Currency code |
| `address.shipping_lines_override` | string | Shipping override |
| `address.include.payment_methods` | array | Payment methods |
| `address.created_at` | string | Created timestamp |
| `address.updated_at` | string | Updated timestamp |

### Subscription Object

| Property | Type | Description |
|----------|------|-------------|
| `subscription.id` | number | Unique ID |
| `subscription.address_id` | number | Address ID |
| `subscription.customer_id` | number | Customer ID |
| `subscription.status` | string | ACTIVE, CANCELLED, EXPIRED |
| `subscription.product_title` | string | Product name |
| `subscription.variant_title` | string | Variant name |
| `subscription.price` | string | Unit price |
| `subscription.quantity` | number | Quantity |
| `subscription.sku` | string | SKU |
| `subscription.order_interval_frequency` | string | Delivery frequency |
| `subscription.order_interval_unit` | string | day/week/month |
| `subscription.charge_interval_frequency` | string | Charge frequency |
| `subscription.next_charge_scheduled_at` | string | Next charge date |
| `subscription.shopify_product_id` | number | Shopify product ID |
| `subscription.shopify_variant_id` | number | Shopify variant ID |
| `subscription.properties` | array | Line item properties |
| `subscription.address` | object | Parent address |

### Customer Object

| Property | Type | Description |
|----------|------|-------------|
| `customer.id` | number | Unique ID |
| `customer.hash` | string | Customer hash (URL identifier) |
| `customer.email` | string | Email |
| `customer.first_name` | string | First name |
| `customer.last_name` | string | Last name |
| `customer.shopify_customer_id` | string | Shopify customer ID |
| `customer.number_subscriptions` | number | Total subscriptions |
| `customer.number_active_subscriptions` | number | Active subscriptions |
| `customer.has_card_error_in_dunning` | boolean | Payment issue flag |
| `customer.can_add_payment_method` | boolean | Can add new payment |
| `customer.tax_exempt` | boolean | Tax exempt |

### Settings Object

Contains `customer_portal` settings:
- `subscription`: add_product, cancel_subscription, change_product, change_quantity, change_variant, edit_order_frequency, edit_scheduled_date, skip_scheduled_order, reactivate_subscription, cancellation_minimum_order_count, cancellation_reason_optional, cancellation_enable_pause_options
- `onetime`: enabled, available_products, shopify_collection_id
- `membership`: allow_membership_cancellation_after
- `discount_input`, `edit_shipping_address`, `hosted_customer_portal`, `view_order_schedule`, `view_recharge_payment_methods`, `show_credits`
- `custom_code`: header, footer, header_logo_url, backend_portal

### Store Object

| Property | Type | Description |
|----------|------|-------------|
| `store.id` | number | Store ID |
| `store.name` | string | Store name |
| `store.domain` | string | Store domain |
| `store.my_shopify_domain` | string | myshopify.com domain |
| `store.currency` | string | Default currency |
| `store.enabled_presentment_currencies` | array | Multi-currency codes |
| `store.enabled_presentment_currencies_symbols` | array | Currency symbols with positioning |
| `store.checkout_platform` | string | "recharge" or "shopify" |
| `store.external_platform` | string | "shopify" |
| `store.payment_processor` | string | "stripe", etc. |
| `store.timezone` | string | Store timezone |
| `store.iana_timezone` | string | IANA timezone |
| `store.bundles_enabled` | boolean | Bundles feature flag |
| `store.test_mode` | boolean | Test mode flag |

---

## Jinja Template Syntax

```jinja
{# Variable output #}
{{ customer.first_name }}

{# Loops #}
{% for subscription in subscriptions %}
  {{ subscription.product_title }} - ${{ subscription.price }}
{% endfor %}

{# Conditionals #}
{% if subscription.status == "ACTIVE" %}
  Active
{% elif subscription.status == "CANCELLED" %}
  Cancelled
{% endif %}

{# URL variables #}
{{ subscription_list_url }}
{{ address_url }}
{{ show_subscription_url | replace('<int:subscription_id>', subscription.id | string) }}

{# Filters #}
{{ subscription.price | money }}
{{ subscription.next_charge_scheduled_at | date("%-m/%-d/%Y") }}
```

---

## Template Files

| File | Page |
|------|------|
| `subscriptions.html` | Subscription list |
| `subscription.html` | Subscription detail |
| `subscription_new.html` | Add subscription |
| `subscription_cancel.html` | Cancel flow |
| `addresses.html` | Address list |
| `address.html` | Address detail |
| `address_new.html` | New address |
| `orders.html` | Order list |
| `order.html` | Order detail |
| `schedule.html` | Delivery schedule |
| `customers/show.html` | Customer info |

---

## Affinity Customer Portal

The newer Affinity portal provides no-code customization with extension points:

- **Events system** for custom logic injection
- **Custom content zones** for announcements, callouts, widgets
- **Custom CSS** for visual branding
- **Advanced configuration** for behavior overrides
- Requires **Recharge Plus** or **Custom** plan

Key Affinity customization docs:
- Events: https://docs.rechargepayments.com/docs/leveraging-events-in-affinity
- Custom content: https://docs.rechargepayments.com/docs/adding-custom-content-to-affinity
- CSS: https://docs.rechargepayments.com/docs/using-custom-css-in-the-affinity-customer-portal
- Configuration: https://docs.rechargepayments.com/docs/using-advanced-configuration-options-with-affinity
