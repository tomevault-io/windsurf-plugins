---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Context

This is **Stock2 (JS Admin)** - the modern replacement for the legacy **SMS (Stock Management System)**. We are migrating a 15-year-old CakePHP application to NodeJS/ReactJS, module by module.

| System | Name | URL | Technology |
|--------|------|-----|------------|
| Legacy | SMS / Stock System | stock.uksoccershop.com | CakePHP 1.x / PHP 5.6 |
| New | Stock2 / JS Admin | stock2.uksoccershop.com | Fastify + React + TypeScript |

**Migration Approach**: Both systems run in parallel. Features are migrated incrementally. Database is shared.

## Documentation scope

This repository holds **code and code-adjacent docs only**: module READMEs, API contracts, OpenAPI specs, dev setup, deployment mechanics. Synthesis docs, audits, RCAs, integration rationale, decision matrices, and historical-decision write-ups are maintained in a separate documentation repository — they do not belong here. If you are drafting one of those, raise it before opening a PR so it can be redirected to the right home.

## Context Loading (Skills)

For migration work, load relevant context from `.claude/skills/`:

| Skill File | When to Load | Keywords |
|------------|--------------|----------|
| `migration-context.md` | Starting migration work, cron jobs | SSO, migration strategy, phases, cron jobs, CLI |
| `sms-reference.md` | Finding SMS documentation | modules, controllers, documentation index |
| `sms-database.md` | Working with existing tables | tables, schema, relationships |
| `marketplace-integration.md` | Marketplace integrations, new marketplace | Fruugo, Sportdeal, OnBuy, feeds, jsa_MarketplaceListing |
| `coding-guidelines.md` | Code style and conventions | patterns, naming, style |
| `marketplace-orders-integration-guide.md` | Adding new marketplace order integrations | orders, fetch, process, status update |
| `marketplace-price-inventory-sync.md` | Adding new marketplace price & inventory sync | sync, price, inventory, adapter, currency, eBay, Allegro, Debenhams |

**SMS Documentation Path**: `D:\wamp64\www\UKSS-SMS-Production\documentation\`

## Migration Priority

**Cron Job Migration Order** (only enabled jobs, CLI-based implementation):
1. **Orders Integration** - Shopify, Baselinker, Fruugo, OnBuy, Walmart, Toffs, MPlaza orders
2. **Product & Inventory Feeds** - Shopify inventory/price, Baselinker sync, marketplace feeds
3. **Remaining Jobs** - FC integration, NP table, BraveOtter, payments, monitoring

See `.claude/skills/migration-context.md` for complete job listings and schedules.

## Project Structure

```
apps/api/         # Fastify API server (TypeScript)
apps/web/         # React frontend (Vite + Shadcn/ui)
packages/shared/  # Shared types, schemas, utilities
e2e/              # Playwright E2E tests
.claude/skills/   # Context loading for AI agents
```

## Common Commands

```bash
# Development
pnpm dev              # Start all dev servers (API: 4000, Web: 3002)
pnpm build            # Build all packages
pnpm lint             # Lint all code

# Database (PostgreSQL default)
pnpm db:push          # Push schema to database
pnpm db:seed          # Seed initial data
pnpm db:studio        # Open Prisma Studio

# Database (MySQL - for SMS compatibility)
DATABASE_PROVIDER=mysql pnpm db:push
DATABASE_PROVIDER=mysql npx tsx apps/api/prisma/seeds/seed-mysql.ts

# Testing
pnpm test:e2e         # Run Playwright E2E tests
pnpm test:e2e:ui      # E2E with interactive UI

# Docker
docker-compose -f docker-compose.dev.yml up -d
docker-compose -f docker-compose.dev.yml down
```

## Architecture

### Tech Stack
- **Frontend**: React 18, Vite, Shadcn/ui, TanStack Query, Zustand, React Hook Form + Zod
- **Backend**: Fastify, Prisma ORM, PostgreSQL/MySQL, Redis, BullMQ
- **Auth**: JWT with access (memory) + refresh (HTTP-only cookie) tokens

### Backend Module Pattern

**Standard modules:**
```
modules/example/
├── example.routes.ts    # Route definitions & handlers
├── example.service.ts   # Business logic (optional)
└── example.schema.ts    # Zod validation schemas
```

**Note:** Marketplace modules use hyphenated naming (`fruugo-routes.ts`, `sportdeal-routes.ts`). The `marketplaces` module has a complex structure with `adapters/`, `cli/`, `config/`, `db/`, `utils/` subdirectories.

### CLI Infrastructure

Cron jobs are implemented as CLI commands using `commander`:
```bash
pnpm cli <command>         # Run CLI commands (e.g., fetch-shopify-orders)
pnpm sync:marketplace      # Sync marketplace inventory
pnpm sync:all-stores       # Sync all stores
```

| Path | Purpose |
|------|---------|
| `apps/api/src/cli.ts` | CLI entry point (commander setup) |
| `apps/api/src/cli/` | CLI command files (fetch-*-orders, sync-*, update-*-status) |
| `ecosystem.config.cjs` | PM2 production process manager config |

Route handlers use Fastify plugins for auth:
```typescript
// Auth only (no specific permission required):
fastify.get('/', {
  preHandler: [fastify.authenticate],
  handler: async (request, reply) => { ... }
})

// Auth + permission check (colon-separated resource:action):
fastify.get('/', {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [velsof/ukss-sms-js-admin](https://github.com/velsof/ukss-sms-js-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
