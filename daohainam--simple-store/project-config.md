---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

SimpleStore is a small e-commerce sample built on **.NET 10** and orchestrated with **.NET Aspire**. The monolith → microservices migration is now complete:

- **Catalog** is a standalone minimal-WebApi service (`SimpleStore.Catalog.API`) that owns `catalogdb` (Postgres). Storefront browsing is anonymous; admin write endpoints require a JWT with the `Admin` role.
- **Identity** (`SimpleStore.Identity.API`) owns `identitydb` (Postgres) and issues JWT bearer tokens (HS256) plus refresh tokens. Web and Admin call it over HTTP for register/login/passkey/profile.
- **Order** (`SimpleStore.Order.API`) owns `orderdb` (Postgres). Storefront endpoints (`/api/v1/order/orders`) require the caller's JWT; admin endpoints (`/api/v1/order/admin/...`) require the `Admin` role.
- **Cart** (`SimpleStore.Cart.API`) is backed by **Redis** (`cart-redis`). It is the first non-DB-backed microservice. Anonymous browsers identify a cart with the opaque `ss_cart` HttpOnly cookie issued by `SimpleStore.Web`, which it passes to Cart.API as the `X-Cart-Id` header; authenticated callers are keyed by the JWT `sub` claim. A `/api/v1/cart/merge` endpoint (auth required) folds the anonymous cart into the user's cart on login.
- **Inventory** (`SimpleStore.Inventory.API`) is the first **event-sourced + CQRS** service and, since v8, the **single source of truth for stock**. The write side appends domain events to **KurrentDB** (formerly EventStoreDB; same open-source product, renamed in 2025); the read side is `inventorydb` (Postgres), populated asynchronously by a `BackgroundService` projector. Admin HTTP surface under `/api/v1/inventory` tracks delivery notes (stock OUT) and receipt notes (stock IN). In v8 it also has a `Reservation` aggregate: it consumes `ReserveStockRequestedEventV1` from the checkout saga, checks projected stock, and publishes `StockReservedEventV1` / `StockReservationFailedEventV1`. Its projector publishes `StockLevelChangedEventV1` whenever stock changes.
- **Checkout** (`SimpleStore.Checkout.API`) is the v8 **MassTransit Saga State Machine** orchestrator. It owns `checkoutdb` (Postgres, saga state only — no business data) and has **no HTTP surface and no JWT** (pure RabbitMQ consumer). Since v12 it drives the create-order → reserve-stock → **process-payment** → confirm/cancel flow, and on a payment failure it runs a **compensation** (release the reserved stock) before cancelling.
- **Payment** (`SimpleStore.Payment.API`) owns `paymentdb` (Postgres) and is the v12 addition. It holds a prepaid **account** (one balance per user, auto-provisioned at zero) plus a transaction ledger. Customers deposit funds (Web "Wallet" page); admins can top up on a customer's behalf (Admin "Payments" page). It consumes `ProcessPaymentRequestedEventV1` from the checkout saga and publishes `PaymentSucceededEventV1` (sufficient balance, debited) or `PaymentFailedEventV1` (insufficient) — making the **account balance the controllable gate** that decides whether a checkout confirms or cancels. User endpoints (`/api/v1/payment/account/...`) require the caller's JWT; admin endpoints (`/api/v1/payment/admin/...`) require the `Admin` role.

**Event-driven flows** ride a **RabbitMQ** broker via **MassTransit**. Shared event records live in `SimpleStore.Contracts` (references only `MassTransit.Abstractions` for the `[MessageUrn]` attribute, added in v11). Every event has a `Vn`-suffixed CLR type, an `int Version` field, and a pinned `MessageUrn`; see [docs/versioning.md](docs/versioning.md). The wired flows:
- **Checkout saga (v8, payment step added v12)** — Order.API publishes `OrderSubmittedEventV1` (carries a `Guid CorrelationId`); Checkout.API's saga consumes it, publishes `ReserveStockRequestedEventV1`; Inventory.API consumes that and publishes `StockReservedEventV1` or `StockReservationFailedEventV1`. On `StockReserved` the saga (v12) moves to `AwaitingPayment` and publishes `ProcessPaymentRequestedEventV1`; Payment.API replies `PaymentSucceededEventV1` → saga publishes `OrderConfirmedEventV1`, or `PaymentFailedEventV1` → saga moves to `CompensatingStock` and publishes `StockReservationCancelRequestedEventV1` (the **compensation**); Inventory releases the hold and replies `StockReservationCancelledEventV1`, after which the saga publishes `OrderCancelledEventV1`. Order.API consumes the confirm/cancel events to set `Order.Status`. Two 30 s timeouts (Quartz, persistent ADO store in `checkoutdb` — survive restarts) bound the stock and payment waits; the payment timeout also runs the stock-release compensation.
- **Stock release / compensation (v12)** — the saga publishes `StockReservationCancelRequestedEventV1`; Inventory.API's `CancelReservationRequestedConsumer` rehydrates the reservation aggregate, appends `StockReservationCancelledV1` to KurrentDB, and its projector adds the held quantity back to `stock_levels.OnHand`, then publishes `StockReservationCancelledEventV1` (back to the saga) + `StockLevelChangedEventV1` (cache refresh). Implements the reservation-cancel the v8 docs marked "deferred to v9".

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daohainam/simple-store](https://github.com/daohainam/simple-store) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
