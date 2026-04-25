---
trigger: always_on
description: The Supabase MCP server is connected to project `bhwyqqbovcjoefezgfnq` (a DIFFERENT project). The v2 app uses `cwsyhpiuepvdjtxaozwf`. **NEVER use MCP for v2 database operations.** Use `curl` with the service role key from `.env.local`, or `psql` with the connection string. If you catch yourself running `mcp__supabase__execute_sql` for v2 data, STOP — you're querying the wrong database.
---

# Goods on Country

## CRITICAL — Read First

### Supabase MCP is the WRONG project
The Supabase MCP server is connected to project `bhwyqqbovcjoefezgfnq` (a DIFFERENT project). The v2 app uses `cwsyhpiuepvdjtxaozwf`. **NEVER use MCP for v2 database operations.** Use `curl` with the service role key from `.env.local`, or `psql` with the connection string. If you catch yourself running `mcp__supabase__execute_sql` for v2 data, STOP — you're querying the wrong database.

### Build, Don't Plan
Do NOT enter extended planning modes unless explicitly asked. When given a task, start implementing immediately. If clarification is needed, ask a focused question — do not write multi-phase plan documents. If a plan IS requested, keep it to bullet points and ask for approval before continuing.

### Verify Approach Before Implementing
Before writing code for a non-trivial task, state your approach in 2-3 bullets: (1) what you'll do, (2) what tools/APIs you'll use, (3) what could go wrong. Then proceed unless redirected. This is NOT a plan — it's a 3-line sanity check.

## What This Is
A social enterprise delivering quality furniture to remote Indigenous communities across Australia. The flagship product is the **Stretch Bed** — a washable, flat-packable bed made from recycled plastic, heavy-duty canvas, and galvanised steel.

## Project Structure

```
v2/                          <- THE ACTIVE CODEBASE (Next.js 16 + React 19)
  src/app/                   <- App Router pages
  src/components/            <- auth, cart, empathy-ledger, layout, marketing, shop, ui
  src/lib/                   <- cart, data, empathy-ledger, ghl, stripe, supabase, types
  public/video/              <- Background videos (hero, stretch-bed, community)
deploy/                      <- OLD static HTML site (legacy, do not modify)
tools/                       <- FFmpeg video processing scripts
media/                       <- Raw video footage and analysis
```

**Always work in `v2/`.** The `deploy/` folder is the old site and should not be modified.

## Tech Stack (v2)
- **Framework:** Next.js 16.1.4 with App Router, Turbopack
- **UI:** React 19, Tailwind CSS 4, Radix UI, shadcn/ui components
- **Backend:** Supabase (PostgreSQL, Auth, Storage) — project `cwsyhpiuepvdjtxaozwf`
- **Payments:** Stripe (only for Stretch Bed purchases)
- **Deployment:** Vercel
- **Fonts:** Georgia (display), system sans-serif (body)

## Products — THE TRUTH

### 1. The Stretch Bed (FLAGSHIP — for sale)
- **What it is:** A flat-packable, washable bed
- **Materials:** Recycled HDPE plastic panels (legs), 2x galvanised steel poles (26.9mm OD × 2.6mm wall), heavy-duty Australian canvas (sleeping surface)
- **How it works:** Two steel poles thread through canvas sleeves. Recycled plastic legs click onto the poles. Done.
- **Specs:** 26kg, 200kg capacity, 188×92×25cm, assembles in ~5 mins, no tools, 10+ year design lifespan, 5-year warranty
- **Plastic:** 20kg of HDPE diverted per bed
- **Manufacturing:** On-country plant — collect plastic, shred, melt, press into bed components. This plant can move to community ownership.
- **Canonical data:** `v2/src/lib/data/products.ts` is the single source of truth for all product specs
- **Supabase slug:** `stretch-bed` (canonical). Shop page URL is still `/shop/stretch-bed-single`.
- **Ecommerce:** Buy now with Stripe. This is the ONLY product for direct sale.

### 2. Washing Machines (PROTOTYPE — register interest)
- **What it is:** Pakkimjalki Kari — named in Warumungu language by Elder Dianne Stokes
- **Base:** Commercial-grade Speed Queen
- **Status:** Prototype stage, deployed in several communities, collecting feedback
- **Goal:** Reduce price point while maintaining reliability for remote conditions
- **Context:** One Alice Springs provider sells $3M/yr of washing machines into remote communities — most end up in dumps within months
- **Ecommerce:** Register Interest form only. Not for direct sale yet.

### 3. Basket Bed (ARCHIVED — open source)
- **What it is:** The first prototype bed — collapsible baskets with zip ties and toppers
- **Variants:** Single, double, stackable
- **Status:** Discontinuing sales. Open-sourcing the design documentation.
- **Ecommerce:** Download Plans link. Not for sale.

### 4. Weave Bed (DISCONTINUED — remove all references)
- **What it was:** A test design that was never produced at scale
- **Action:** Remove any remaining references when found. The Supabase `product_type: weave_bed` and slugs like `weave-bed-*` are WRONG — these should be `stretch_bed` and `stretch-bed-*`.

## Key Patterns

### Component Organisation
```
src/components/
  ui/          -> shadcn/ui primitives (Button, Card, Dialog, etc.)
  marketing/   -> Hero, ImpactStats, ProductCard
  layout/      -> SiteHeader, SiteFooter
  shop/        -> ProductDetail, AddToCart
  empathy-ledger/ -> FeaturedStories, CommunityGallery
```

### Data Layer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Acurioustractor) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
