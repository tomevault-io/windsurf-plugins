---
trigger: always_on
description: In-app purchase webhook handling for Rails. Provides a Rails engine with webhook endpoint, paywall helpers, JavaScript for Hotwire Native bridge communication, and optional Pay gem integration.
---

# purchasekit gem

In-app purchase webhook handling for Rails. Provides a Rails engine with webhook endpoint, paywall helpers, JavaScript for Hotwire Native bridge communication, and optional Pay gem integration.

## Architecture

```
PurchaseKit (this gem)
├── Configuration - API credentials, demo mode, event handlers
├── Engine - Rails engine with controllers, helpers, JavaScript
├── ApiClient - HTTP client for PurchaseKit SaaS API
├── Product - Product abstraction (demo/remote)
├── Purchase::Intent - Purchase intent abstraction (demo/remote)
├── Events - Event publishing and callback system
├── WebhookSignature - HMAC-SHA256 signature verification
├── Pay integration (conditional) - Auto-detected when Pay gem is present
└── Error classes
```

## Pay gem integration

The gem auto-detects the Pay gem via `PurchaseKit.pay_enabled?` (checks `defined?(::Pay)`).

**With Pay gem**: Webhooks automatically create Pay::Subscription records and broadcast Turbo Stream redirects via ActionCable.

**Without Pay gem**: Use event callbacks to handle subscriptions with your own models.

## Rails engine

The gem provides a Rails engine that mounts at `/purchasekit`:

```ruby
# config/routes.rb
mount PurchaseKit::Engine, at: "/purchasekit"
```

This adds:
- `POST /purchasekit/webhooks` - Receives webhooks from PurchaseKit SaaS
- `POST /purchasekit/purchases` - Creates purchase intents for native apps (responds with Turbo Stream append)
- `POST /purchasekit/purchase/completions/:id` - Demo mode only - called by iOS after Xcode StoreKit purchase completes

### Controllers

- `PurchaseKit::WebhooksController` - Verifies signature, publishes events via callback system, queues for Pay if available
- `PurchaseKit::PurchasesController` - Creates intents, appends response div via Turbo Stream (form stays visible but disabled)
- `PurchaseKit::Purchase::CompletionsController` - Demo mode only, skips CSRF (called directly by iOS), publishes events and redirects
- `PurchaseKit::ApplicationController` - Base controller with `rescue_from` for `NotFoundError` (404) and `SubscriptionRequiredError` (402)

### Helpers

The `purchasekit_paywall` helper renders a paywall form:

```erb
<% pay_customer = current_user.set_payment_processor(:purchasekit) %>
<%= purchasekit_paywall customer_id: pay_customer.id, success_path: dashboard_path do |paywall| %>
  <%= paywall.plan_option product: @annual, selected: true do %>
    Annual - <%= paywall.price %>/year
  <% end %>
  <%= paywall.submit "Subscribe" %>
  <%= paywall.restore %>
<% end %>
```

Builder methods: `plan_option`, `price`, `submit`, `restore`. The `restore` method accepts an optional `url:` parameter. When provided, the JS controller POSTs subscription IDs to the URL after reading them from StoreKit/Play Billing. Without `url:`, it dispatches a `purchasekit--paywall:restore` DOM event for custom handling.

`purchasekit_paywall` accepts an optional `proration_mode:` (default `"charge_prorated_price"`) that controls how Google Play handles base plan swaps within one umbrella subscription (for example monthly to annual). It is passed to the Android bridge and ignored on Apple, which handles intra-group upgrades automatically. See `android/CLAUDE.md` for accepted values.

`customer_id` is whatever identifier you want the webhook to receive back. With Pay, it **must** be `Pay::Customer.id` — `SubscriptionCreated` and `SubscriptionUpdated` both do `Pay::Customer.find(event["customer_id"])`. Without Pay, use your own user ID.

### JavaScript

The gem provides a single unified Stimulus controller for both Pay and non-Pay integrations:

- `purchasekit--paywall` Stimulus controller for Hotwire Native bridge communication
- `purchasekit/turbo_actions` custom Turbo Stream action for redirects

The controller handles prices, purchases, restore, errors, and includes a 30-second fallback redirect if ActionCable isn't connected. The `restore()` action sends a bridge message and dispatches a `purchasekit--paywall:restore` CustomEvent with `{ subscriptionIds, error }` in the detail.

#### Purchase lifecycle events

The controller dispatches a CustomEvent at each transition so host apps can swap their own copy (e.g. "Confirming your purchase...") without guessing timing. Same `this.dispatch(...)` pattern as `restore`, so all are prefixed with the controller identifier:

| Event | Fires when |
|-------|-----------|
| `purchasekit--paywall:initiated` | The purchase intent is created and the native purchase is about to start. Detail: `{ correlationId }` |
| `purchasekit--paywall:store-confirmed` | The native bridge replies with a non-terminal-error status (the store confirmed the purchase). Detail: `{ status }` |
| `purchasekit--paywall:awaiting-webhook` | The form is disabled and waiting for the webhook-driven redirect. Detail: `{}` |
| `purchasekit--paywall:complete` | The redirect fires, either from the broadcast Turbo Stream `redirect` action or the 30-second fallback. Detail: `{}` |

Listen on the paywall element (or a parent, since these bubble):

```javascript
document.addEventListener("purchasekit--paywall:awaiting-webhook", () => {
  // update your copy
})
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PurchaseKit/purchasekit](https://github.com/PurchaseKit/purchasekit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
