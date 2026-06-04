---
trigger: always_on
description: Guidelines for AI-assisted development on the Stripe Payment Connector for Composable Commerce.
---

# CLAUDE.md

Guidelines for AI-assisted development on the Stripe Payment Connector for Composable Commerce.

## Project Overview

This is a **Commercetools Connect** payment connector that integrates Stripe with Commercetools. It consists of two packages:

- **processor** (`/processor`): Node.js backend service (Fastify v5, TypeScript) that handles payment operations, webhook events, and Stripe/CT translation
- **enabler** (`/enabler`): Frontend component library (Vite) wrapping Stripe Payment Element for merchant storefronts

The connector runs inside the Commercetools Connect managed runtime. Users install it via marketplace or import as a custom application. Users configure it through environment variables only -- they do not manage infrastructure.

## Architecture Invariants

These are non-negotiable design rules. Do not violate them.

### Authentication

- **Every route must have a `preHandler` auth hook.** No exceptions. Choose the correct one:
  - `sessionHeaderAuthHook.authenticate()` -- for frontend-facing endpoints (called by merchant storefront via enabler)
  - `oauth2AuthHook.authenticate()` + `authorizationHook.authorize(...)` -- for backend/service-to-service endpoints
  - `jwtAuthHook.authenticate()` -- for internal service endpoints (status, payment-components)
  - `stripeHeaderAuthHook.authenticate()` -- for webhook endpoint only (paired with `constructEvent()`)
- **`constructEvent()` is the sole cryptographic guard for webhooks.** The `StripeHeaderAuthHook` is a lightweight pre-filter only. This is Stripe's prescribed pattern. Do not attempt to "fix" the hook by adding signature verification there unless you also remove `constructEvent()` from the route (consolidation is acceptable, but both must not verify independently).
- **Raw body must be preserved for webhook routes.** The `fastify-raw-body` plugin is configured for `/stripe/webhooks` specifically. If adding new webhook routes, ensure they are included in the `routes` array.

### Data Flow

- **Stripe is the source of truth for payment state.** The connector translates Stripe events into Commercetools payment transactions -- never the reverse.
- **Commercetools is the source of truth for prices.** Subscription price sync reads prices from CT and updates Stripe, not the other way around.
- **Metadata is the bridge.** Stripe payment intents carry CT identifiers (cart_id, payment_id, customer_id, etc.) in their metadata. This metadata must always be set during creation and must never be modified after.
- **Webhook events drive state updates.** Payment/order state changes happen via webhook processing (eventual consistency), not via synchronous API responses.

### Payment Flow Integrity

- **This connector uses Stripe's deferred intent pattern.** Elements are mounted with `mode` + `amount` + `currency` but NO `clientSecret`. The PaymentIntent is created server-side at submit time (`POST /payments`), not at page load or component mount. This means no PI exists until the customer commits to paying.
- **Cart freeze is the integrity guarantee.** When a PI is created, the cart is frozen immediately after. A frozen cart cannot be modified (no item additions, quantity changes, or shipping updates). This guarantees that `PI amount === cart total` for the lifetime of the PI. Do not add code paths that break this invariant.
- **Amount validation is intentionally not hardcoded in the webhook handler.** In automatic capture, PI amount always equals the cart total (guaranteed by cart freeze). In manual capture, multi-capture, incremental auth, and extended auth, the captured amount legitimately differs from the original authorization. A hard amount check would break these flows. The existing warn on unfrozen cart at `payment_intent.succeeded` is the correct signal for anomalies.
- **Do not add amount validation that blocks order creation** unless it is scoped to `capture_method === 'automatic'` only.

### Route Flow Separation

- **Shipping routes (`/shipping-methods/*`) are Express Checkout infrastructure only.** They exist to support the Apple Pay / Google Pay sheet's shipping address and rate selection lifecycle. The enabler wires them exclusively to `StripeExpressCheckoutElement` events (`shippingaddresschange`, `shippingratechange`, `cancel`). Payment Element flows never call these routes.
- **`GET /shipping-methods/remove` is the Express Checkout cancellation handler.** It unfreezes the cart because the Express Checkout flow was abandoned. In the deferred intent pattern, no PI has been created at the time of cancellation (PI creation only happens on `confirm`, which is mutually exclusive with `cancel`). The unfreeze is correct behavior -- it releases the cart for further shopping.
- **`getShippingMethods` and `updateShippingRate` temporarily unfreeze and re-freeze.** This handles the case where the cart may be frozen from a prior flow. These methods always restore the original frozen state after the shipping update.
- **Do not add general-purpose unfreeze endpoints.** Cart freeze/unfreeze is tied to specific payment lifecycle events, not exposed as a standalone operation.

### Configuration

- **Secrets go in `securedConfiguration` in `connect.yaml`.** Never in `standardConfiguration`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stripe/stripe-commercetools-connect-app](https://github.com/stripe/stripe-commercetools-connect-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
