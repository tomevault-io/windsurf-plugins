---
trigger: always_on
description: Enterprise ecommerce NuGet for Umbraco v17+.
---

# Merchello Agent Guide

## Mission
Enterprise ecommerce NuGet for Umbraco v17+.
Ethos: making enterprise ecommerce simple; avoid over-engineered code.

## Read First (CRITICAL)
1. Trace real code paths before changing anything.
- Do not assume docs are correct.
- Start from controller to service to factory/provider, then modify.

2. Preserve architecture boundaries.
- Controllers: HTTP orchestration only; no business logic; no `DbContext`.
- Services: business logic and data access.
- Factories: all domain object creation; never `new Entity {}` directly.

3. Keep single source of truth calculations centralized.
- Never duplicate business math in controllers, handlers, views, or JS.
- Must call designated services/providers:

| Concern | Source of truth |
| --- | --- |
| Basket totals | `CheckoutService.CalculateBasketAsync()` |
| Tax rate lookup | `TaxService.GetApplicableRateAsync()` |
| Shipping tax proportional math | `ITaxCalculationService.CalculateProportionalShippingTax()` |
| Payment status | `PaymentService.CalculatePaymentStatus()` |
| Inventory reserve/allocate/release/reverse | `InventoryService` |
| Shipping base cost fallback chain | `ShippingCostResolver.ResolveBaseCost()` |
| Shipping quote retrieval | `IShippingQuoteService.GetQuotes*()` |

4. Multi-currency invariants are strict.
- Basket amounts are stored in store currency and do not change on display currency changes.
- Display uses multiply: `amount * rate`.
- Checkout/payment (invoice creation) uses divide: `amount / rate`.
- Lock rate at invoice creation (`PricingExchangeRate`, source, timestamp).
- Never charge from display amounts.

5. Shipping tax invariants are strict.
- Never hardcode shipping tax rates.
- Always query provider:
  - `ITaxProviderManager.IsShippingTaxedForLocationAsync()`
  - `ITaxProviderManager.GetShippingTaxRateForLocationAsync()`
- Return semantics:
  - `0m`: explicitly not taxed
  - `> 0`: use specific rate
  - `null`: use proportional shipping tax calculation
- Required entry points: checkout calculation, storefront display context, invoice shipping tax recalculation.

6. Persistence/serialization pitfalls to avoid.
- SQLite EF projection aggregates: do not use `Min()`/`Max()` in projected SQL for SQLite. Use in-memory group aggregation pattern.
- `Dictionary<string, object>` + `System.Text.Json`: always call `UnwrapJsonElement()` before `Convert.*`.
- `EFCoreScope`: do not begin nested transactions inside `ExecuteWithContextAsync()`.

7. Idempotency and dedupe must remain intact.
- Payments: preserve `Payment.IdempotencyKey` and `Payment.WebhookEventId` behavior.
- Fulfilment/webhook paths: use duplicate webhook checks where provided.

8. Notification ordering matters.
- Lower priority runs first.
- Keep fault tolerance in handlers: catch/log and do not rethrow.

## Core Architecture
### Layering Rules
- Modular plugin system via `ExtensionManager`.
- Service methods should use parameter models (RORO style).
- Return `CrudResult<T>` for mutations that can fail.
- Use `async/await` and propagate `CancellationToken`.
- Add DB tables only when necessary.

### Feature Structure
```text
Feature/
  Dtos/        # API transfer objects
  Extensions/  # C# extension methods
  Factories/
  Mapping/
  Models/      # Internal domain
  Services/
    Parameters/
    Interfaces/
```
Create folders only when needed.

## Domain Invariants
### Products and Variants
- `ProductOption.IsVariant` defaults to `true` and controls variant generation.
- If `IsVariant = false`, treat as add-on using `PriceAdjustment`, `CostAdjustment`, `SkuSuffix`.
- Maintain `ProductRoot` vs `Product` responsibilities:
  - `ProductRoot`: parent-level config (`TaxGroupId`, option definitions, default package config).
  - `Product`: variant-level SKU, price, stock, package overrides.

### Inventory and Warehouses
- Stock lifecycle for tracked inventory:
  - Reserve: `Reserved += qty`
  - Allocate (ship): `Stock -= qty`, `Reserved -= qty`
  - Cancel/release: `Reserved -= qty`
- Warehouse selection order:
  1. `ProductRootWarehouse` priority
  2. service region eligibility
  3. stock availability (`Stock - Reserved >= qty`)

### Checkout, Shipping, and Grouping
- `IShippingService.GetShippingOptionsForBasket()` is the basket-level entry point and uses order grouping internally.
- Shipping cost priority: `State -> Country -> Universal(*) -> FixedCost`.
- Flat-rate and dynamic providers are different:
  - Flat-rate uses configured destination/fixed costs.
  - Dynamic providers (`UsesLiveRates = true`) fetch rates live and must not rely on fixed-cost entries.
- Dynamic provider visibility depends on provider enablement and warehouse config.
- `ProductRoot.AllowExternalCarrierShipping = false` must block dynamic carrier options for those products.

Shipping selection key contract (must remain stable):
- Flat-rate: `so:{guid}`
- Dynamic: `dyn:{provider}:{serviceCode}`
- Parse this contract into order fields (`ShippingProviderKey`, `ShippingServiceCode`, `ShippingServiceName`) and honor quoted rates from checkout session.

### Tax
Product tax:
- Preserve `TaxGroupId` from product root through line items into orders/invoice tax payloads.
- External tax providers depend on this mapping for tax code selection.

Shipping tax:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YodasMyDad/Merchello](https://github.com/YodasMyDad/Merchello) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
