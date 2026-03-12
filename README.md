# Recharge Skill for Claude Code

A Claude Code skill for the [Recharge Subscriptions](https://developer.rechargepayments.com/2021-11) platform (Shopify).

## What it does

Provides Claude with full knowledge of the Recharge API to help you:

- Re-sync subscription data
- Reinitialize / reset webhooks
- Retry failed charges and refresh billing state
- Reconnect payment methods
- Diagnose and fix Recharge integration issues
- Work with the Theme Engine customer portal API
- Use the Storefront JS SDK (`@rechargeapps/storefront-client`)

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
- "Reset subscription billing"

## Contents

| File | Description |
|------|-------------|
| `SKILL.md` | Main skill — common operations & quick reference |
| `references/api_reference.md` | All Admin API endpoints |
| `references/webhooks.md` | Webhook topics, payloads, reinit guide |
| `references/troubleshooting.md` | Diagnostic tree + fixes |
| `references/theme_and_sdk.md` | Theme Engine portal API + Storefront SDK |

## Author

[@amoyano](https://github.com/amoyano)
