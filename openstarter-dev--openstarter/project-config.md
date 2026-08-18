---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

openstarter is a production-ready full-stack SaaS starter built with:
- **Monorepo**: Turborepo + pnpm workspaces
- **Framework**: TanStack Start + TanStack Router (web)
- **Backend**: Hono RPC API mounted at `/api/*`
- **Database**: Drizzle ORM (SQLite/Turso/Postgres/MySQL)
- **Multi-platform**: Web, Desktop (Electron), Mobile (Expo), Browser Extension (WXT), CLI, Mini-App (Taro)
- **Auth**: Better-Auth with OAuth, passkey, 2FA, organizations
- **Billing**: Stripe/PayPal/Alipay/WeChat Pay with credit system
- **i18n**: inlang/Paraglide
- **Testing**: Vitest + fast-check
- **Linting**: Biome

## Monorepo Structure

```
openstarter/
├── apps/                  # Client applications
│   ├── web/               # TanStack Start (SSR, file-based routing)
│   ├── desktop/           # Electron + React
│   ├── mobile/            # Expo Router (React Native)
│   ├── extension/         # WXT + React
│   ├── cli/               # Command-line interface
│   └── mini-app/          # Taro (WeChat Mini App)
│
├── packages/              # Shared libraries (published as @openstarter/*)
│   ├── api/               # Hono RPC backend (see API Architecture below)
│   ├── auth/              # Better-Auth server + clients (web.ts, mobile.ts)
│   ├── billing/           # Billing logic (shared, web, mobile providers)
│   ├── db/                # Drizzle schema, migrations, seed scripts
│   ├── email/             # React Email templates + providers (Resend, Cloudflare)
│   ├── i18n/              # Translations + locale utils
│   ├── shared/            # Constants, validators, utilities, logger
│   ├── storage/           # S3/R2 storage abstractions
│   ├── analytics/         # Event tracking (web, mobile, extension variants)
│   ├── monitoring/        # Error tracking (web, mobile, extension variants)
│   ├── notifications/     # Notification providers
│   └── ui/                # shadcn components, Tailwind (web, mobile variants)
│
└── tooling/               # ESLint, Prettier, TypeScript, Vitest configs
```

## API Architecture (`packages/api/`)

The API is organized as modular domains under `src/modules/`, with each domain following a consistent structure:

```
src/
├── index.ts               # Hono app entry, route composition root
├── middleware/            # Auth, RBAC, plan gating middleware
├── schema/                # Shared Zod validation schemas (pagination, id param)
└── modules/               # Domain modules (routers + services colocated)
    ├── ai/                # AI generation (Replicate/Fal providers)
    ├── ai-tasks/          # AI task lifecycle + credit deduction
    ├── auth/              # Better-Auth passthrough + custom unlink-account
    ├── billing/           # Checkout, payment webhooks
    ├── config/            # Public config endpoint
    ├── storage/           # Image upload (S3/R2)
    ├── user/              # Profile, subscriptions, credits, orders
    ├── admin/             # Admin-only routes
    │   ├── rbac/          # Role/permission management
    │   ├── analytics/     # Admin metrics
    │   ├── overview/      # User/order/subscription/credit lists, invite codes
    │   └── tickets/       # Support ticket management
    ├── support/           # User-facing support
    │   ├── tickets/       # Create/reply to support tickets
    │   └── apikeys/       # API key management
    ├── content/           # Blog, posts, taxonomy, SEO
    │   ├── posts/         # Article CRUD
    │   ├── blog/          # Published articles list
    │   ├── taxonomy/      # Categories/tags
    │   └── seo/           # SEO data (sitemap, llms.txt)
    └── demo/              # Demo endpoints (notes, private-data)
```

### Module Pattern

Each domain module follows this pattern:
```
modules/{domain}/
├── router.ts              # Hono router (includes route definitions + validation)
├── service.ts             # Business logic (optional, for complex domains)
└── index.ts               # Barrel export (optional)
```

For larger modules with sub-domains:
```
modules/{domain}/
├── router.ts              # Domain aggregator router
├── {subdomain}/
│   ├── router.ts          # Sub-router
│   └── service.ts         # (optional)
└── ...
```

### Schema Organization

Shared schemas are centralized in `src/schema/shared.ts`:
- `idParam` — standard ID parameter schema
- `paginationSchema` — default pagination (page + pageSize)
- `createPaginationSchema(max, default)` — custom pagination

Routers extend these with domain-specific fields:
```ts
const listQuery = paginationSchema.extend({
  status: z.enum(STATUS_VALUES).optional(),
  search: z.string().optional(),
});
```

## Commands

| Task | Command |
| --- | --- |
| **Install deps** | `pnpm install` |
| **Dev (all/specific)** | `pnpm dev` / `pnpm --filter web dev` |
| **Build (all/specific)** | `pnpm build` / `pnpm --filter api build` |
| **Lint / Format** | `pnpm lint` / `pnpm format` |
| **Lint fix / Format fix** | `pnpm lint:fix` / `pnpm format:fix` |
| **Test (all/specific)** | `pnpm test` / `pnpm --filter @openstarter/api test` |
| **Test coverage** | `pnpm test:coverage` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openstarter-dev/openstarter](https://github.com/openstarter-dev/openstarter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
