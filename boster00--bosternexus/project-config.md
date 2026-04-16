---
trigger: always_on
description: You are an expert full-stack developer working on **Boster Nexus**, an internal operations and enablement web app for Bosterbio. This is a Next.js application built on the ShipFast boilerplate foundation.
---

You are an expert full-stack developer working on **Boster Nexus**, an internal operations and enablement web app for Bosterbio. This is a Next.js application built on the ShipFast boilerplate foundation.

## 🎯 Project Purpose

Boster Nexus centralizes Boster's cross-system workflows across:
- **Zoho Books** (source of truth for transactional data)
- **Zoho CRM** (customer relationship management)
- **Zoho Desk** (support tickets)
- **Magento** (storefront/e-commerce)
- **Marketing Analytics** (GA4/BigQuery)

The app acts as a **cache/warehouse** and **system of record for derived/internal-only data**, not replacing source systems but enabling efficient operations and guardrails.

## 🚀 Tech Stack

- **Next.js**: 15.1.9 with React 19 and Turbopack
- **React**: 19.0.0 (latest stable)
- **Tailwind CSS**: 4.0.0 (CSS-based configuration)
- **DaisyUI**: 5.0.50 (latest with new component system)
- **Supabase**: Latest SSR package with async cookies support (Auth + Postgres)
- **Database**: Managed Postgres (Neon/Supabase) - primarily cache/warehouse
- **Deployment**: Vercel

## 📊 Core Data Principles

### Source of Truth Hierarchy
1. **Zoho Books** is the source of truth for:
   - Orders, invoices, POs, bills, inventory state
   - All transactional financial data
2. **Nexus DB** stores:
   - **Raw entity caches** (structured tables from Books/CRM/Desk)
   - **Derived/computed state** that Zoho cannot represent cleanly:
     - Bill↔PO line matching
     - Guardrail exceptions
     - Dashboard layout config
     - Loyalty ledger
     - Wiki versions
3. **BigQuery** remains the store for GA4-scale event data; Nexus pulls only what it needs on-demand

### Rate Limiting Constraints
⚠️ **CRITICAL**: Zoho API rate limit is **no more than 1 request per second**
- The Zoho class (`@/libs/zoho.js`) handles rate limiting automatically
- **Always use the Zoho class** - never make direct API calls
- For long chains, prefer async jobs (Postgres job table)
- Design must prevent parallel overload
- The Zoho class ensures all requests respect the 1 req/sec limit

## 📁 File Structure & Organization

### Route Groups (Next.js App Router)
```
app/
├── (public)/              # Public pages (no auth required)
│   ├── layout.js         # Public layout wrapper
│   ├── page.js           # Landing page
│   ├── blog/             # Blog pages
│   ├── privacy-policy/   # Legal pages
│   └── tos/
├── (private)/            # Private pages (auth required)
│   ├── layout.js         # Private layout with auth check
│   ├── admin/            # Admin functions
│   ├── freezer/          # Freezer label printer
│   └── zoho-test/        # Zoho testing interface
├── api/                  # API routes
│   ├── zoho/            # Zoho webhook ingestion and operations
│   ├── auth/            # Authentication endpoints
│   ├── stripe/          # Stripe payment endpoints
│   ├── migrations/       # Database migration endpoints
│   └── webhook/         # Webhook handlers
├── signin/              # Authentication pages
├── dashboard/           # Dashboard pages
└── layout.js            # Root layout
```

### Service Layer Architecture
```
libs/
├── zoho/                # Zoho integration
│   ├── index.js         # CRITICAL: Central Zoho class for all Zoho operations
│   ├── auth.js          # Zoho authentication service
│   ├── config.js        # Zoho configuration
│   ├── rate-limiter.js  # Rate limiting for Zoho API
│   ├── entities/        # Entity classes for data transformation
│   │   ├── books/       # Zoho Books entities
│   │   └── index.js
│   ├── orchestration/   # Orchestration services
│   │   └── ZohoOrchestrator.js
│   └── services/        # Business logic services
│       └── books/       # Zoho Books services
├── supabase/            # Supabase client utilities
│   ├── client.js        # Client-side Supabase client
│   ├── server.js        # Server-side Supabase client
│   ├── data-access-layer.js  # CRITICAL: Data Access Layer for all DB operations
│   └── middleware.js    # Supabase middleware utilities
├── utils/               # Shared utilities
│   └── logger.js        # Structured file logging utility
├── zoho.js              # Legacy export (re-exports from zoho/index.js)
├── stripe.js            # Stripe integration
├── api.js               # API client utilities
└── [other]/             # Other utility libraries
```

### Database Organization
```
Database Tables:
- zoho_books_*           # Zoho Books data (e.g., zoho_books_items, zoho_books_invoices, zoho_books_salesorders, zoho_books_purchaseorders, zoho_books_bills)
- zoho_books_*_items     # Line items (e.g., zoho_books_invoice_items, zoho_books_salesorder_items)
- auth_tokens            # OAuth tokens for Zoho and other services
- profiles               # User profiles
- derived_*               # Computed/derived state (e.g., derived_bill_po_matches)
- jobs_*                 # Async job queue (e.g., jobs_sync, jobs_reconciliation)
- config_*               # Configuration (e.g., config_dashboard_layouts, config_report_definitions)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/boster00) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
