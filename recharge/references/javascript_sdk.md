# Recharge JavaScript SDK Reference

Complete reference for `@rechargeapps/storefront-client` — the official Recharge JavaScript SDK for building custom storefronts and customer portals. Use this skill when building headless Shopify frontends with Recharge, implementing subscription widgets, managing customer subscriptions from the client side, or integrating Recharge into Hydrogen/custom storefronts.

**Docs:** https://storefront.rechargepayments.com/client/

## Overview

| Feature | Details |
|---------|---------|
| Package | `@rechargeapps/storefront-client` |
| Distribution | NPM (CJS & ESM) + CDN (script tag) |
| Runtime | Browser (client-side) + Node.js (server-side) |
| TypeScript | Fully typed |
| Auth | Recharge Storefront Access Token (`strfnt_...`) |
| Session | 1-hour expiry, caller manages storage |

## Installation

### Package (NPM/Yarn)

```bash
npm install @rechargeapps/storefront-client
# or
yarn add @rechargeapps/storefront-client
```

### Script Tag (CDN)

```html
<script src="https://static.rechargecdn.com/storefront-client/latest/storefront-client.umd.js"></script>
```

When using UMD, all methods are available on `window.rechargeStorefrontClient`.

---

## Initialization

Must be called before any other SDK method.

### ESM

```typescript
import { initRecharge } from '@rechargeapps/storefront-client';

initRecharge({
  storeIdentifier: 'mystore.myshopify.com',
  storefrontAccessToken: 'strfnt_...',
  appName: 'my-app',
  appVersion: '1.0.0',
  loginRetryFn: async () => {
    const session = await loginShopifyAppProxy();
    return session;
  },
});
```

### UMD

```javascript
window.rechargeStorefrontClient.initRecharge({
  storeIdentifier: 'mystore.myshopify.com',
  storefrontAccessToken: 'strfnt_...',
  appName: 'my-app',
  appVersion: '1.0.0',
});
```

| Parameter | Required | Description |
|-----------|----------|-------------|
| `storeIdentifier` | Yes (headless) | myshopify.com domain |
| `storefrontAccessToken` | Yes | Token starting with `strfnt_` |
| `appName` | No | Your app identifier |
| `appVersion` | No | Your app version |
| `loginRetryFn` | No | Called on 401 to auto-refresh session |

**Token creation:** Recharge merchant admin -> API tokens -> type "Storefront"

---

## Authentication Methods

All API methods (except auth) require a `session` object containing `apiToken` and optionally `customerId`. Sessions expire after 1 hour.

### Shopify Theme Storefront

```typescript
import { loginShopifyAppProxy } from '@rechargeapps/storefront-client';
const session = await loginShopifyAppProxy();
```

### Headless with Shopify Storefront API

```typescript
import { loginWithShopifyStorefront } from '@rechargeapps/storefront-client';
const session = await loginWithShopifyStorefront(
  shopifyStorefrontToken,
  shopifyCustomerAccessToken
);
```

### Headless with Shopify Customer Account API

```typescript
import { loginWithShopifyCustomerAccount } from '@rechargeapps/storefront-client';
const session = await loginWithShopifyCustomerAccount(shopifyCustomerAccessToken);
```

### Recharge Customer Portal Context

```typescript
import { loginCustomerPortal } from '@rechargeapps/storefront-client';
const session = await loginCustomerPortal();
```

### Passwordless (Email/SMS Code)

```typescript
import { sendPasswordlessCode, validatePasswordlessCode } from '@rechargeapps/storefront-client';

// Step 1: Send code
const { session_token } = await sendPasswordlessCode('user@email.com', {
  send_email: true,
  send_sms: true
});

// Step 2: Validate code
const session = await validatePasswordlessCode('user@email.com', session_token, 'code_from_email');
```

For Shopify App Proxy context, use `sendPasswordlessCodeAppProxy` and `validatePasswordlessCodeAppProxy`.

### Session Messages

Login functions return a `message` field:
- `Success` — Customer logged in
- `Failed to validate customer with Shopify.` — Customer doesn't exist in Shopify
- `Error occurred in call to Shopify.` — Shopify error
- `Customer does not exist in Recharge.` — Customer not in Recharge
- `No Shopify customer access token given.` — Missing token
- `Invalid Recharge storefront access token.` — Bad SDK init token

### Auth Scopes

Scopes on the Storefront Token determine what the session can access:
- `read products` and `read product search` are always available if enabled (no customer login needed)
- Customer-scoped methods (`read customers`, `read subscriptions`, `write subscriptions`, etc.) require authenticated customer session
- All merchant-portal customer portal settings are honored by the SDK

---

## API Methods (17 Categories)

### Addresses

| Method | Scope | Description |
|--------|-------|-------------|
| `listAddresses(session, params?)` | read customers | List customer addresses |
| `getAddress(session, id)` | read customers | Get address by ID |
| `createAddress(session, data)` | write customers | Create new address |
| `updateAddress(session, id, data)` | write customers | Update address |

```typescript
import { listAddresses, createAddress, updateAddress } from '@rechargeapps/storefront-client';

const addresses = await listAddresses(session);
const newAddr = await createAddress(session, {
  address1: '123 Main St',
  city: 'Portland',
  province: 'Oregon',
  country_code: 'US',
  zip: '97201',
  first_name: 'Jane',
  last_name: 'Doe',
  phone: '5551234567'
});
await updateAddress(session, newAddr.id, { address2: 'Suite 200' });
```

### Bundles (API)

| Method | Scope | Description |
|--------|-------|-------------|
| `listBundles(session, params?)` | read subscriptions | List bundles |
| `getBundleSettings(session, productId)` | read subscriptions | Get bundle config |

### Bundle Selections (API)

| Method | Scope | Description |
|--------|-------|-------------|
| `listBundleSelections(session, params?)` | read subscriptions | List selections |
| `getBundleSelection(session, id)` | read subscriptions | Get selection |
| `createBundleSelection(session, data)` | write subscriptions | Create selection |
| `updateBundleSelection(session, id, data)` | write subscriptions | Update selection |
| `deleteBundleSelection(session, id)` | write subscriptions | Delete selection |

### Charges

| Method | Scope | Description |
|--------|-------|-------------|
| `listCharges(session, params?)` | read orders | List charges |
| `getCharge(session, id)` | read orders | Get charge by ID |
| `applyDiscountToCharge(session, id, code)` | write orders | Apply discount |
| `removeDiscountsFromCharge(session, id)` | write orders | Remove discounts |
| `skipCharge(session, id, purchaseItemIds)` | write orders | Skip charge items |
| `unskipCharge(session, id, purchaseItemIds)` | write orders | Unskip charge items |
| `processCharge(session, id)` | write payments | Process charge |
| `rescheduleCharge(session, id, date)` | write payments | Reschedule charge |

```typescript
import { listCharges, skipCharge, processCharge } from '@rechargeapps/storefront-client';

const charges = await listCharges(session, { limit: 25, sort_by: 'scheduled_at-asc' });
await skipCharge(session, 123, [1234, 2345]);
await processCharge(session, 456);
```

### Collections

| Method | Scope | Description |
|--------|-------|-------------|
| `listCollectionProducts(session, collectionId, params?)` | read products | Products in collection |

### Credits

| Method | Scope | Description |
|--------|-------|-------------|
| `listCredits(session, params?)` | read credits | List customer credits |

### Customer

| Method | Scope | Description |
|--------|-------|-------------|
| `getCustomer(session)` | read customers | Get current customer |
| `updateCustomer(session, data)` | write customers | Update customer |
| `getDeliverySchedule(session, params?)` | read customers | Upcoming deliveries |

```typescript
import { getCustomer, getDeliverySchedule } from '@rechargeapps/storefront-client';

const customer = await getCustomer(session);
const schedule = await getDeliverySchedule(session, {
  future_interval: 90,
  date_min: '2024-01-01'
});
```

### Gifts

| Method | Scope | Description |
|--------|-------|-------------|
| `listGifts(session, params?)` | read subscriptions | List gifts |

### Metafields

| Method | Scope | Description |
|--------|-------|-------------|
| `listMetafields(session, params)` | read on owner | List metafields |
| `getMetafield(session, id)` | read on owner | Get metafield |
| `createMetafield(session, data)` | write on owner | Create metafield |
| `updateMetafield(session, id, data)` | write on owner | Update metafield |
| `deleteMetafield(session, id)` | write on owner | Delete metafield |

### Onetimes

| Method | Scope | Description |
|--------|-------|-------------|
| `listOnetimes(session, params?)` | read subscriptions | List one-times |
| `getOnetime(session, id)` | read subscriptions | Get one-time |
| `createOnetime(session, data)` | write subscriptions | Create one-time |
| `updateOnetime(session, id, data)` | write subscriptions | Update one-time |
| `deleteOnetime(session, id)` | write subscriptions | Delete one-time |

### Orders

| Method | Scope | Description |
|--------|-------|-------------|
| `listOrders(session, params?)` | read orders | List orders |
| `getOrder(session, id)` | read orders | Get order by ID |

### Payment Methods

| Method | Scope | Description |
|--------|-------|-------------|
| `listPaymentMethods(session, params?)` | read payment_methods | List methods |
| `getPaymentMethod(session, id)` | read payment_methods | Get method |
| `updatePaymentMethod(session, id, data)` | write payment_methods | Update method |

### Plans

| Method | Scope | Description |
|--------|-------|-------------|
| `listPlans(session, params?)` | read plans | List selling plans |

### Products

| Method | Scope | Description |
|--------|-------|-------------|
| `searchProducts(session, params)` | read product_search | Search products |

### Store

| Method | Scope | Description |
|--------|-------|-------------|
| `getStore(session)` | - | Store info (no customer auth needed) |

### Subscriptions

| Method | Scope | Description |
|--------|-------|-------------|
| `listSubscriptions(session, params?)` | read subscriptions | List subscriptions |
| `getSubscription(session, id)` | read subscriptions | Get subscription |
| `createSubscription(session, data)` | write subscriptions | Create subscription |
| `updateSubscription(session, id, data)` | write subscriptions | Update subscription |
| `cancelSubscription(session, id, data)` | write subscriptions | Cancel subscription |
| `activateSubscription(session, id)` | write subscriptions | Reactivate |
| `updateSubscriptionChargeDate(session, id, date)` | write subscriptions | Change charge date |
| `updateSubscriptionAddress(session, id, addressId)` | write subscriptions | Move to address |
| `skipSubscriptionCharge(session, id, date)` | write orders | Skip upcoming charge |
| `skipGiftSubscriptionCharge(session, ids, recipient)` | write subscriptions | Gift & skip |
| `createSubscriptions(session, subs, opts?)` | write subscriptions | Bulk create (max 20) |
| `updateSubscriptions(session, addressId, subs, opts?)` | write subscriptions | Bulk update (max 20) |

```typescript
import {
  listSubscriptions,
  createSubscription,
  updateSubscription,
  cancelSubscription,
  activateSubscription,
  skipSubscriptionCharge
} from '@rechargeapps/storefront-client';

// List
const subs = await listSubscriptions(session, { limit: 25, sort_by: 'id-asc' });

// Create
await createSubscription(session, {
  address_id: 75875888,
  charge_interval_frequency: 30,
  next_charge_scheduled_at: '2024-10-18',
  order_interval_frequency: 1,
  order_interval_unit: 'day',
  quantity: 1,
  external_variant_id: { ecommerce: '31589634277439' },
  external_product_id: { ecommerce: '4378133856319' },
});

// Update
await updateSubscription(session, 123, { quantity: 2 });

// Change charge date
await updateSubscriptionChargeDate(session, 123, '2024-10-18');

// Move to different address
await updateSubscriptionAddress(session, 123, 456);

// Cancel
await cancelSubscription(session, 123, {
  cancellation_reason: 'Do not want it anymore.',
  send_email: true,
});

// Reactivate
await activateSubscription(session, 123);

// Skip upcoming charge by date
await skipSubscriptionCharge(session, 123, '2024-10-24');

// Gift subscription charge
await skipGiftSubscriptionCharge(session, [123], {
  email: 'friend@example.com',
  address1: '456 Oak Ave',
  city: 'Omaha',
  country_code: 'US',
  first_name: 'Friend',
  last_name: 'Name',
  phone: '123456789',
  province: 'Nebraska',
  zip: '68144',
});

// Bulk create (max 20, same address)
await createSubscriptions(session, subscriptionsArray, { allow_onetimes: true });

// Bulk update (max 20, same address)
await updateSubscriptions(session, addressId, subscriptionsArray, { allow_onetimes: true });
```

---

## CDN Methods (3 Categories)

Data from Recharge CDN is cached by the SDK. Use `resetCDNCache()` to force re-fetch.

### Subscription Widget

| Method | Description |
|--------|-------------|
| `getSubscriptionWidget()` | Get widget settings (colors, labels, form type) |

### Products (CDN)

| Method | Description |
|--------|-------------|
| `getProduct(productId)` | Get product with plans and variants |
| `getProducts()` | Get all products |

### Bundles (CDN)

| Method | Description |
|--------|-------------|
| `getBundleProduct(productId)` | Get bundle product configuration |

### CDN URLs

Base: `https://static.rechargecdn.com/store/{store_identifier}`

| File | Path |
|------|------|
| Single product | `/product/2021-08/{external_product_id}.json` |
| All products | `/product/2021-08/products.json` |
| Widget settings | `/2021-08/widget_settings.json` |
| Store settings | `/2021-08/store_settings.json` |

```typescript
import { getProduct, resetCDNCache } from '@rechargeapps/storefront-client';

const product = await getProduct('4771');
// product.plans[], product.variants[], product.external_product_id

resetCDNCache(); // clear cache to get fresh data
```

---

## Bundle Methods (Storefront)

These methods work only within the Shopify storefront domain (browser context).

### Static Bundles

```typescript
import { getBundleId, getBundleSettings } from '@rechargeapps/storefront-client';

const settings = await getBundleSettings(productId);
// settings.variants[].option_sources - collections for product selection
// settings.variants[].items_count - number of items per variant

const bundle = {
  externalVariantId: '123',
  externalProductId: '456',
  selections: [{
    collectionId: '789',
    externalProductId: '111',
    externalVariantId: '222',
    quantity: 3
  }]
};

const bundleId = await getBundleId(bundle);
// Add bundleId as line item property in cart
```

### Dynamic Bundles

```typescript
import { getDynamicBundleItems } from '@rechargeapps/storefront-client';

const bundle = {
  externalVariantId: '123',
  externalProductId: '456',
  selections: [{
    collectionId: '789',
    externalProductId: '111',
    externalVariantId: '222',
    quantity: 3,
    // SCI stores:
    sellingPlan: 12345,
    // RCS stores:
    shippingIntervalFrequency: 30,
    shippingIntervalUnitType: 'day',
    discountedVariantId: 999
  }]
};

const items = await getDynamicBundleItems(bundle);
// Add items to cart
```

---

## Common Patterns

### Session Management with Retry

```typescript
import { initRecharge, loginShopifyAppProxy } from '@rechargeapps/storefront-client';

let currentSession = null;

initRecharge({
  storeIdentifier: 'mystore.myshopify.com',
  storefrontAccessToken: 'strfnt_...',
  loginRetryFn: async () => {
    currentSession = await loginShopifyAppProxy();
    return currentSession;
  },
});

currentSession = await loginShopifyAppProxy();
```

### Headless Hydrogen Integration

```typescript
import { initRecharge, loginWithShopifyCustomerAccount } from '@rechargeapps/storefront-client';

initRecharge({
  storeIdentifier: 'mystore.myshopify.com',
  storefrontAccessToken: 'strfnt_...',
});

const session = await loginWithShopifyCustomerAccount(shopifyAccessToken);
```

### Error Handling

```typescript
try {
  const subscription = await getSubscription(session, 123);
} catch (error) {
  if (error.status === 401) {
    // Session expired, re-authenticate
    const newSession = await loginShopifyAppProxy();
    const subscription = await getSubscription(newSession, 123);
  } else if (error.status === 404) {
    // Subscription not found
  } else {
    console.error('Recharge SDK error:', error);
  }
}
```

### Subscription Widget Data Flow

```typescript
import { getProduct, getSubscriptionWidget } from '@rechargeapps/storefront-client';

const widget = await getSubscriptionWidget();
const product = await getProduct(shopifyProductId);

// Build subscription options from product.plans
const plans = product.plans.map(plan => ({
  id: plan.id,
  label: plan.title,
  interval: `${plan.order_interval_frequency} ${plan.interval_unit}`,
  discount: plan.discount_amount,
  type: plan.type
}));

// Use widget settings for styling
const { active_color, background_color, font_color, subscribe_message } = widget;
```
