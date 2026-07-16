---
trigger: always_on
description: Instructions building apps with MCP
---


# KAUVEX COMMERCE CLOUD (KCC)
# Version: 2.0

## Platform
- Name: KAUVEX — "Everything. Everywhere. Delivered."
- Framework: Next.js 14 App Router + TypeScript
- Styling: Tailwind CSS + shadcn/ui
- ORM: Prisma
- Database: PostgreSQL (Supabase)
- Auth: Supabase Auth
- Storage: Supabase Storage
- Hosting: Vercel
- Colors: Navy #0A1628 | Orange #FF6B00
- Font: Inter

## Architecture
- Multi-storefront: path | subdomain | custom domain
- Multi-vendor: unlimited vendors with plan tiers
- Multi-warehouse: FBK + merchant fulfilled
- Centralized: one DB, one admin, one vendor login

## Key Directories
- /prisma/schema.prisma — Full database schema (~375 models)
- /lib/affiliates/ — Affiliate & Influencer Network (tracking, commission, payouts, fraud, onelink, promotions, b2b)
- /app/partners/ — Partner portal (associate/influencer registration, dashboard, links, tools, analytics)
- /app/influencer/ — Influencer storefront builder, product picker, promo manager
- /app/admin/affiliates/ — Admin full control panel (partners, commissions, payouts, fraud, promotions, b2b)
- /prisma/seeds/roles.ts — RBAC seed script
- /lib/permissions.ts — RBAC permission system
- /lib/storefront-context.tsx — Storefront context provider
- /lib/storefront-resolver.ts — Server-side storefront resolution
- /lib/buybox.ts — Buy box engine with weighted scoring
- /lib/search-engine.ts — Client search utilities
- /lib/security.ts — Rate limiting, 2FA, audit logging, validation
- /lib/ai/ — AI feature modules (descriptions, SEO, recommendations)
- /lib/shipping/ — Carrier integrations (dhl, fedex, aramex, local, gig, kwik, dhl-express-international, fedex-international, aramex-international, freight-forwarder)
- /lib/packaging-engine.ts — Packaging material selection, tier logic, compliance, order packaging records
- /lib/logistics-warehouse.ts — Warehouse staff portal (pick tasks, pack tasks, inbound receiving, inventory)
- /lib/logistics/ — Logistics engine (dispatch.ts, shipping-engine.ts, partner-tiers.ts, delivery-tiers.ts, fbk-debt.ts, terminology.ts)
- /lib/fuel/ — Fuel management engine (surcharge.ts, data-service.ts)
- /components/logistics/ShipmentTimeline.tsx — Unified cross-tier tracking display
- /app/express/ — Kauvex Express public courier (landing, book, track, business)
- /app/logistics/ — Partner portal (register, login, dashboard)
- /app/admin/logistics/ — Full admin control panel (rates, payouts, packaging, map, insurance, gaps, fbk, express)
- /app/admin/shipping/ — Admin shipping management (zones, surge-pricing, restrictions, hs-codes, business-accounts)
- /app/vendor/shipping/profiles/ — Profile builder for vendor shipping rules
- /app/vendor/shipping/dropoff/ — Drop-off manifest system
- /app/admin/packaging/ — Packaging materials master registry
- /app/admin/logistics/jobs/ — Admin logistics jobs management
- /app/admin/logistics/warehouses/ — Admin warehouse locations
- /app/admin/cj-dropshipping/packaging/ — CJ Dropshipping packaging config
- /app/express/carbon/ — Carbon footprint tracker (Phase 23)
- /app/express/corporate/ — Corporate & B2B services (Phase 23)
- /app/express/rates/calendar/ — Smart rate calendar (Phase 23)
- /app/express/delivery-confidence/ — Delivery confidence score (Phase 23)
- /app/logistics/why-kauvex/ — Competitor comparison page (Phase 23)
- /app/vendor/logistics/ — Vendor logistics section (shipments, pickups, manifests, performance, packaging guide)
- /app/vendor/fbk/packaging/ — FBK packaging tier configuration
- /app/supplier/logistics/ — Supplier packaging + delivery management
- /app/logistics/fleet/ — Logistics partner fleet management
- /app/warehouse/ — Warehouse staff portal (dashboard, inbound, outbound, inventory, packaging-stock, reports)
- /app/orders/[id]/tracking/ — Customer order tracking with live GPS
- /app/track/[trackingNumber]/ — Public Express tracking page
- /app/api/v1/express/ — Express API routes (waybills, pricing, tracking)
- /app/api/v1/logistics/ — Logistics API routes (tracking, partners, jobs, payouts)
- /app/api/v1/shipping/insurance/ — Insurance reserve API
- /app/api/v1/shipping/packaging/ — Packaging elements API
- /app/api/v1/shipping/customs/ — Customs document generation API
- /lib/cart-recovery.ts — Abandoned cart recovery engine
- /lib/bundles.ts — Product bundle management
- /lib/catalog-mode.ts — Catalog mode for B2B storefronts
- /lib/vendor-metrics.ts — Vendor health scoring
- /lib/api-helpers.ts — REST API response utilities
- /lib/validators/ — Zod validation schemas
- /components/home/ — Homepage section components (8 sections)
- /components/search/ — Voice search + barcode scanner
- /app/admin/ — Admin panel routes (60+ pages: commerce, sales, marketing, marketplace, operations, system)
- /app/admin/analytics/ — Analytics dashboards (realtime, search, BI)
- /app/vendor/ — Vendor panel routes
- /app/vendor/store-builder/ — Store builder with plan-gated features
- /app/vendor/fbk/ — FBK enrollment and management
- /app/vendor/advertising/ — Ad campaign manager
- /app/api/v1/ — REST API v1 (17 route groups)
- /app/vendor/inventory/ — Full inventory management (FBK + merchant)
- /app/vendor/inventory/replenishment-alerts/ — Reorder threshold alerts
- /app/vendor/products/add/ — Catalog matching entry point

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shuddi1962/kauvex](https://github.com/shuddi1962/kauvex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
