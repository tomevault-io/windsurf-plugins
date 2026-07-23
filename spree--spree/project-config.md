---
trigger: always_on
description: validates :name, presence: true
---

# Spree Commerce — Development Rules

## Plans & Architecture Decisions

All feature plans live in `docs/plans/` using the template at `docs/plans/_template.md`. Never create plans elsewhere.

When proposing significant architectural changes:
1. Check existing plans in `docs/plans/` for conflicts
2. Create or update a plan using the template before implementing
3. Pay special attention to "Constraints on Current Work" sections — these apply even when you're not implementing that plan directly

Use `/project:create-plan` and `/project:update-plan` for plan management.

Active plans (6.0 target, work pending):
- `6.0-multi-vendor-marketplace.md` — Open-source the marketplace core (Vendor, OrderGroup-based order splitting, commission engine with EU commission taxation, `VendorTransfer`/`VendorPayout` ledger + pluggable `PayoutProvider`) per spree/spree#13323. 6.0 headline feature; rebuilds the legacy Enterprise multi-vendor module as native models on the Cart/Order split. Basic Stripe Connect payouts (Express onboarding + on-fulfillment transfers) ship OSS in the monorepo, alongside the Stripe core gateway pulled in from the standalone `spree_stripe` repo (payment-sessions classes only, likely `spree/core` — decisions.md 2026-07-15); Enterprise keeps refund clawbacks/netting, reconciliation, KYC ops, DAC7 payout reports, facilitator taxes + Shopify/WooCommerce vendor apps.
- `6.0-cart-order-split.md` — Cart/Order model separation, polymorphic LineItem
- `6.0-admin-api.md` — Admin REST API conventions, auth, endpoint list (~300 endpoints)
- `6.0-admin-spa.md` — React admin architecture, extension points, table registry, i18n + server-error mapping
- `6.0-product-types.md` — Prototype → ProductType rename, MetafieldDefinition schema enforcement
- `6.0-remove-master-variant.md` — Eliminate is_master, add default_variant_id FK on Product
- `6.0-split-adjustments.md` — Replace polymorphic Adjustment with TaxLine, Discount, Fee
- `6.0-typed-stock-movements.md` — Replace generic StockMovement with typed kinds + concrete FKs
- `6.0-normalize-state-to-status.md` — Rename state → status on Payment, Shipment, InventoryUnit, ReturnAuthorization, GiftCard
- `6.0-fulfillment-and-delivery.md` — Shipment→Fulfillment, ShippingMethod→DeliveryMethod, drop ShippingCategory, FulfillmentProvider strategy, pickup (merchant StockLocation) + pickup_point (third-party PickupPointProvider)
- `6.0-returns-exchanges-claims.md` — First-class Return, Exchange, Claim models replacing ReturnAuthorization/Reimbursement chain
- `6.0-platform-auth.md` — Drop Devise, own auth stack, User→Customer/Staff rename (RefreshToken shipped in 5.4)
- `6.0-tax-provider.md` — Per-Market TaxProvider, replaces TaxRate.adjust + Calculator, drop Zone model (TaxRate gets direct country/state FKs)
- `6.0-delivery-rate-provider.md` — Per-DeliveryMethod DeliveryRateProvider, replaces Estimator + Calculator, DeliveryZone with postal code support
- `6.0-rich-text-descriptions.md` — Drop ActionText storage, store HTML in text columns, sanitize on write, serve `description` + `description_html` in API (description_html serializer field shipped in 5.4)
- `6.0-inventory-operations.md` — StockTransfer lifecycle (draft → ready_to_ship → in_transit → received with partial receive), new `Spree::PurchaseOrder` + `Spree::Supplier` (renamed from Vendor — `Spree::Vendor` is the marketplace seller, see decisions.md 2026-07-14) replacing today's "external receive" hack, variant + stock-location stock history panels. Consumes the typed-movement primitives from `6.0-typed-stock-movements.md`.
- `6.0-replace-taxons-with-categories.md` — Split Taxon into Category (hierarchy) + Collection (flat/rule-based). `Spree::Category < Spree::Taxon` alias + Category API surface shipped in 5.5; table rename + Collection pending.

Multi-version plans (some phases shipped, some pending):
- `5.4-store-api-naming-standardization.md` — Standardize API naming against industry (address fields, discounts, customer_note, label, brand/last4, etc.). 5.4 model/API aliases shipped; 6.0 column/table renames pending.
- `5.4-6.0-eu-legal-compliance.md` — GDPR (data export/anonymization, consent timestamps), Omnibus (PriceHistory, lowest-in-30-days), Consumer Rights (withdrawal period). 5.4 PriceHistory + `prior_price` shipped; GDPR endpoints + withdrawal period still pending.
- `5.4-6.0-custom-fields-rename.md` — Rename Metafields → Custom Fields. 5.4 API bridge + 5.5 `Spree::CustomField`/`CustomFieldDefinition` constant aliases shipped; 6.0 model/table rename pending.
- `5.4-6.0-product-media-system.md` — Product-level media gallery. 5.5 data model (spree_variant_media, media_type, focal_point, external_video_url) shipped; admin UIs in progress; 6.0 cleanup pending.
- `5.5-6.0-order-cancellation-and-approval.md` — First-class `OrderCancellation` + `OrderApproval` models. 5.5 models + migrations shipped; 6.0 drops denormalized columns.
- `5.5-6.0-display-on-to-boolean.md` — Collapse `display_on` tri-state to a single `storefront_visible` boolean. 5.5 bridge (`storefront_visible` accessor + Ransacker on `Spree::DisplayOn`) shipped; 6.0 schema rename pending.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spree/spree](https://github.com/spree/spree) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
