# Recharge Skill for Claude Code

A comprehensive Claude Code skill for the [Recharge Subscriptions](https://developer.rechargepayments.com/2021-11) platform (Shopify).

## What it does

Provides Claude with full knowledge of the Recharge platform across four key areas:

### Admin API (`references/api_reference.md`)
- All REST endpoints (Subscriptions, Charges, Customers, Addresses, Orders, Payment Methods, Plans, Discounts, Webhooks, Metafields, Onetimes, Bundle Selections, Async Batches)
- Authentication, scopes, cursor pagination, sorting, response extending
- Platform compatibility tags (RCS, SCI, Custom, BigCommerce)
- HTTP status codes and rate limiting

### Theme Engine (`references/theme_engine.md`)
- Customer portal customization with Jinja templates
- Request Objects (schema-based data fetching)
- All portal routes (addresses, subscriptions, charges, payment methods, orders, onetimes, discounts, memberships)
- Template objects (address, subscription, customer, settings, store)
- Affinity customer portal extension points

### JavaScript SDK (`references/javascript_sdk.md`)
- `@rechargeapps/storefront-client` package (NPM + CDN)
- All authentication methods (App Proxy, Shopify Storefront, Customer Account, Passwordless, Customer Portal)
- 17 API method categories (Subscriptions, Charges, Addresses, Bundles, etc.)
- CDN methods for product/widget data
- Bundle methods for static and dynamic bundles

### Webhooks (`references/webhooks.md`)
- All 25+ webhook topics with descriptions
- Payload enrichment via `included_objects`
- Full reset/reinitialization procedures
- Programmatic webhook management

### Troubleshooting (`references/troubleshooting.md`)
- Diagnostic decision tree
- Fixes for charges, subscriptions, payments, auth, orders, addresses
- Platform-specific notes
- SDK debugging guide

## Install

```bash
npx skills add amoyano/recharge-skill
```

Or manually copy the `recharge/` folder into `~/.claude/skills/`.

## Trigger phrases

- "Recharge is not working"
- "Resync subscriptions"
- "Fix Recharge connection"
- "Reinitialize webhooks"
- "Refresh billing"
- "Recharge API"
- "Recharge theme engine"
- "Customer portal"
- "Recharge SDK"
- "Recharge headless"
- "Failed charges"

## Contents

| File | Description |
|------|-------------|
| `SKILL.md` | Main skill — architecture overview, quick recipes, routing |
| `references/api_reference.md` | Complete Admin API reference (all endpoints, objects, scopes) |
| `references/theme_engine.md` | Theme Engine reference (portal routes, templates, request objects) |
| `references/javascript_sdk.md` | JavaScript SDK reference (storefront-client, auth, methods) |
| `references/webhooks.md` | Webhooks reference (topics, payloads, reset procedures) |
| `references/troubleshooting.md` | Troubleshooting guide (diagnostic tree, common fixes) |

## Author

[@amoyano](https://github.com/amoyano)
