---
trigger: always_on
description: AI working guide for `foysal50x/tashil`. Read before touching code.
---

# CLAUDE.md — Tashil

AI working guide for `foysal50x/tashil`. Read before touching code.

## What this package is

Laravel subscription + feature management. **Owns** plan catalog, subscription state, feature gating, atomic usage counters, trial lifecycle, scheduled state transitions, invoice issuance, the **transaction ledger** (recording the payments/refunds the host executes), the **activate-on-payment → renewal → dunning → reactivation** state machine, **proration on in-place plan changes**, immutable event log, and route middleware for gating. **Does NOT own** payment capture, the actual retry *charge* during dunning, refund execution, gateway sync, or the wallet/balance that funds Metered features — host app handles money movement, reports the result via `Tashil::billing()->record*()`, and binds `MeteredBilling` for metered billing. Read [docs/09-Billing-Lifecycle.md](docs/09-Billing-Lifecycle.md) before touching activation / renewal / dunning / proration code.

**Billing model:** a priced plan (`price > 0`, `requires_payment = true`) subscribes as `Pending` with NO access; an `initial` invoice is issued, and `activate()` runs when it's paid (anchoring the period to `paid_at`). Free / `requires_payment = false` plans subscribe `Active` immediately. Trials subscribe `OnTrial` (access granted) and bill at `convertTrial()`. Gating is decided by the **package's own `requires_payment` flag** — `tashil.billing.activate_on_payment` (default `true`) only seeds that flag at creation when the caller doesn't set it; flipping the config later affects only new packages. Create packages with `requires_payment = false` for the legacy "access first, bill on renewal" model.

Composer: `foysal50x/tashil` · Namespace: `Foysal50x\Tashil` · PHP `^8.2` · Laravel `^10|^11|^12|^13`.

## Authoritative docs

Source of truth for design questions. Read before changing related code.

| Topic | File |
|---|---|
| Tables + ER + indexes | [docs/01-DB-Schema.md](docs/01-DB-Schema.md) |
| Feature types (incl. Metered), snapshot, counter, reset | [docs/02-Feature-System.md](docs/02-Feature-System.md) |
| Trial lifecycle, isOnTrial semantics | [docs/03-Trial-System.md](docs/03-Trial-System.md) |
| Six scheduled commands + idempotency | [docs/04-Scheduler-Jobs.md](docs/04-Scheduler-Jobs.md) |
| Event log, point-in-time replay, analytics | [docs/05-Reporting-Data-Model.md](docs/05-Reporting-Data-Model.md) |
| Layout, conventions, Subscribable, middleware, MeteredBilling | [docs/06-Developer-Guide.md](docs/06-Developer-Guide.md) |

If you change behavior, update the matching doc in the same change.

## Layout

```
src/
  Builders/         PackageBuilder, FeatureBuilder — fluent catalog construction
  Console/          Six scheduled commands (tashil:*)
  Contracts/        Subscribable, MeteredBilling, repository interfaces
  Enums/            SubscriptionStatus, FeatureType (Boolean/Limit/Consumable/Enum/Metered),
                    ResetPeriod, UsageOperation, InvoiceStatus, TransactionStatus, Period
  Events/           Subscription / Trial / Invoice / Usage / Metered events
  Exceptions/       MeteredBillingNotConfiguredException
  Facades/          Tashil
  Http/Middleware/  EnsureSubscribed, EnsurePlan, EnsureFeature
  Managers/         DatabaseManager, CacheManager
  Models/           BaseModel + Eloquent models
  Observers/        InvoiceObserver, TransactionObserver
  Providers/        TashilServiceProvider (binds + middleware + Blade + schedule)
  Repositories/     EloquentX + Cache decorators
  Services/         SubscriptionService, UsageService, BillingService,
                    AnalyticsService, EventStore, Resetter, Generators/,
                    Providers/NullMeteredBilling
  Support/          DateFmt, helpers
  Traits/           HasSubscriptions
  Tashil.php        Facade target — sub-accessors + resolveSubscribable APIs
database/           Single migration + factories
tests/Feature       Integration tests (real DB, time travel)
tests/Unit          Builder / Model / Enum tests
docs/               Authoritative design docs (see table above)
examples/           Cookbook usage
```

## Layering — respect it

```
HasSubscriptions trait (implements Subscribable)
    → Tashil facade → SubscriptionService / UsageService / BillingService / AnalyticsService / EventStore
        → Repository interfaces (Contracts/)
            → Eloquent or Cache-decorated impls
                → Eloquent models → DB
        → MeteredBilling (host-implemented) — called by UsageService for Metered features
        → Subscribable resolver — called by middleware via Tashil::resolveSubscribable
```

Rules:
- Hot, mutating models (`SubscriptionEvent`, `SubscriptionFeature`, `FeatureUsage`) bypass cache. Don't add cache decorators here.
- Cold catalog (`Package`, `Feature`) + analytics flow through cache decorators.
- New persistence concerns go behind an interface in `Contracts/`. Bind in `TashilServiceProvider::register()` so host `boot()` can override.
- Subscriber type-hints across the codebase are `Subscribable`, never Eloquent's `Model`. If you find a stray `Model $subscriber`, fix it.

## Hard invariants — DO NOT break

These are enforced in code AND tested. Breaking them = bug.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Foysal50x/tashil](https://github.com/Foysal50x/tashil) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
