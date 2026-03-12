# Recharge Theme Engine & Storefront SDK

## Overview

| Layer | Use Case |
|-------|----------|
| Theme Engine API | Customer portal in Shopify Liquid theme (`/tools/recurring/portal/`) |
| Storefront SDK (`@rechargeapps/storefront-client`) | Headless / custom frontends |
| Admin API (`api.rechargeapps.com`) | Server-side integration |

---

## Theme Engine API

Base path: `/tools/recurring/portal/<customer_hash>/`
Auth: `?token=<customer_token>` query param (or session)

### Read Data — Request Objects

Fetch multiple resources in one call:

```javascript
const schema = JSON.stringify({
  customer: {},
  subscriptions: [],
  payment_methods: [],
  addresses: [],
  orders: []
});
const url = `/portal/${customerHash}/request_objects?token=${token}&schema=${schema}`;
const data = await fetch(url).then(r => r.json());
```

### Subscriptions

```
GET    /tools/recurring/portal/{hash}/subscriptions
POST   /tools/recurring/portal/{hash}/subscriptions              # Create
POST   /tools/recurring/portal/{hash}/subscriptions/bulk         # Bulk create
GET    /tools/recurring/portal/{hash}/subscriptions/{id}
POST   /tools/recurring/portal/{hash}/subscriptions/{id}         # Update
POST   /tools/recurring/portal/{hash}/subscriptions/{id}/activate
POST   /tools/recurring/portal/{hash}/subscriptions/{id}/cancel
POST   /tools/recurring/portal/{hash}/subscriptions/{id}/skip
POST   /tools/recurring/portal/{hash}/subscriptions/{id}/unskip
POST   /tools/recurring/portal/{hash}/subscriptions/{id}/delay
POST   /tools/recurring/portal/{hash}/subscriptions/{id}/swap    # Swap product
POST   /tools/recurring/portal/{hash}/subscriptions/{id}/charge_date
```

```javascript
// Update subscription
await fetch(`/tools/recurring/portal/${hash}/subscriptions/${id}`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ quantity: 2, order_interval_frequency: "30" })
}).then(r => r.json());

// Activate subscription
await fetch(`/tools/recurring/portal/${hash}/subscriptions/${id}/activate`, {
  method: 'POST'
}).then(r => r.json());
```

### Addresses

```
GET    /tools/recurring/portal/{hash}/addresses
POST   /tools/recurring/portal/{hash}/addresses                  # Create
GET    /tools/recurring/portal/{hash}/addresses/{id}
POST   /tools/recurring/portal/{hash}/addresses/{id}             # Update
POST   /tools/recurring/portal/{hash}/addresses/{id}/charges/skip
POST   /tools/recurring/portal/{hash}/addresses/merge
```

### Charges

```
GET    /tools/recurring/portal/{hash}/charges
POST   /tools/recurring/portal/{hash}/charges/process
POST   /tools/recurring/portal/{hash}/charges/skip
POST   /tools/recurring/portal/{hash}/charges/unskip
```

### Payment Methods (Novum 4+)

```
GET    /tools/recurring/portal/{hash}/payment_methods
GET    /tools/recurring/portal/{hash}/payment_methods/{id}
POST   /tools/recurring/portal/{hash}/payment_methods/{id}       # Update
```

```javascript
// Refresh payment method
await fetch(`/tools/recurring/portal/${hash}/payment_methods/${id}`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ payment_token: 'new_token' })
}).then(r => r.json());
```

### Customer

```
GET    /tools/recurring/portal/{hash}/customer
POST   /tools/recurring/portal/{hash}/customer                   # Update
```

### Orders

```
GET    /tools/recurring/portal/{hash}/orders
GET    /tools/recurring/portal/{hash}/orders/{id}
POST   /tools/recurring/portal/{hash}/orders/{id}/delay
```

### One-Time Products

```
POST   /tools/recurring/portal/{hash}/onetime                    # Create
GET    /tools/recurring/portal/{hash}/onetime/{id}
POST   /tools/recurring/portal/{hash}/onetime/{id}               # Update
POST   /tools/recurring/portal/{hash}/onetime/{id}/charge_date
POST   /tools/recurring/portal/{hash}/onetime/{id}/cancel
```

### Discounts

```
GET    /tools/recurring/portal/{hash}/discounts
POST   /tools/recurring/portal/{hash}/discounts/apply
POST   /tools/recurring/portal/{hash}/discounts/remove
```

### Memberships

```
GET    /tools/recurring/portal/{hash}/memberships
GET    /tools/recurring/portal/{hash}/memberships/{id}
POST   /tools/recurring/portal/{hash}/memberships/{id}/activate
POST   /tools/recurring/portal/{hash}/memberships/{id}/cancel
```

### Utility Endpoints

```
GET    /tools/recurring/portal/{hash}/schedule        # Delivery schedule
GET    /tools/recurring/portal/{hash}/settings        # Portal settings
GET    /tools/recurring/portal/{hash}/shop            # Shop object
GET    /tools/recurring/portal/{hash}/store           # Store object
GET    /tools/recurring/portal/{hash}/products        # Product search
```

---

## Storefront SDK (`@rechargeapps/storefront-client`)

For headless / custom frontend implementations.

### Installation

```bash
npm install @rechargeapps/storefront-client
# or
yarn add @rechargeapps/storefront-client
```

### Initialization

```javascript
import { initRecharge, loginShopifyAppProxy } from '@rechargeapps/storefront-client';

initRecharge({
  storeIdentifier: 'mystore.myshopify.com',  // required for headless
  storefrontAccessToken: 'strfnt_...',         // must start with strfnt
  appName: 'my-app',
  appVersion: '1.0.0',
  loginRetryFn: async () => {
    // Called on 401 — refresh session
    const session = await loginShopifyAppProxy();
    return session;
  },
});
```

**Token:** Create in Recharge merchant admin → API tokens → type "Storefront"

### SDK Method Categories

| Category | Count | Description |
|----------|-------|-------------|
| CDN methods | 3 | CDN-based integration helpers |
| Bundles | 1 | Pre-packaged bundle utilities |
| API methods | 17 | Core API calls (subscriptions, customers, charges, etc.) |

Full methods: https://storefront.rechargepayments.com/client/docs/category/methods/

### Reconnecting / Refreshing Session

```javascript
// Force a session refresh if auth fails
import { loginShopifyAppProxy } from '@rechargeapps/storefront-client';
const session = await loginShopifyAppProxy();
```
