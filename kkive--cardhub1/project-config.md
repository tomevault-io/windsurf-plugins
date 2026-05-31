---
trigger: always_on
description: **Purpose:** A card management platform for creating, browsing, sharing, and trading digital cards. Supports SillyTavern/TavernAI-compatible character card import/export. Includes free and paid content with secure download workflows. Collections bundle exactly 3 cards (character + worldbook + preset) and export as ZIP.
---

# AGENTS.md - Cards hub

## Project

**Name:** Cards hub
**Purpose:** A card management platform for creating, browsing, sharing, and trading digital cards. Supports SillyTavern/TavernAI-compatible character card import/export. Includes free and paid content with secure download workflows. Collections bundle exactly 3 cards (character + worldbook + preset) and export as ZIP.

## Architecture

```
cards-hub/
+-- apps/
|   +-- api/          - NestJS backend (REST API + BullMQ worker for exports, cleanup, stats, search-sync)
|   +-- web/          - Umi Max / Ant Design Pro frontend
+-- packages/
|   +-- shared/       - Shared TypeScript types (card schema, SillyTavern compat)
+-- infra/
|   +-- docker/       - Dockerfiles, nginx config
+-- docker-compose.yml
+-- AGENTS.md         - This file (AI collaboration rules)
+-- README.md         - Human setup guide
```

### Stack

| Layer | Technology |
|-------|-----------|
| API | NestJS 10 + Prisma 6 + MySQL 8.4 |
| Queue | BullMQ + Redis 7 |
| Search | Meilisearch |
| Web | Umi Max 4 + Ant Design 5 |
| Auth | JWT + WebAuthn / Passkeys |
| Container | Docker Compose |
| Worker | BullMQ background worker for exports, cleanup, stats, and search index sync (does not serve user pages) |

### Key Directories

- `apps/api/src/` - NestJS source (controllers, services, modules)
- `apps/api/src/auth/` - Auth module (register, login, JWT, `/api/auth`)
- `apps/api/src/card/` - Card CRUD, search, publish/unpublish, admin list
- `apps/api/src/collection/` - Collection CRUD, publish/unpublish, admin list, ZIP export
- `apps/api/src/order/` - Order creation and admin list
- `apps/api/src/file/` - File upload, download, card export
- `apps/api/src/worker/` - BullMQ worker processors (export, cleanup, stats, search-sync)
- `apps/api/src/worker.ts` - BullMQ worker entrypoint (`pnpm start:worker`)
- `apps/api/prisma/` - Prisma schema, migrations, and seed script
- `apps/web/src/pages/cards/` - Mobile-friendly card marketplace (search, filter drawer, detail drawer, paid order creation)
- `apps/web/src/pages/collections/` - Public collection market (search, detail, whole ZIP export, paid order creation)
- `apps/web/src/pages/admin/` - Mobile admin console: overview, cards, collections, orders, config, audit
- `apps/web/src/pages/login/` - Mobile-safe login page (JWT + passkey)
- `apps/web/src/pages/register/` - Mobile-safe registration page
- `apps/web/src/services/api.ts` - Fetch-based API client (JWT from localStorage, `{ data, error }` shape)
- `apps/web/src/layouts/index.tsx` - Sticky top nav bar with mobile hamburger drawer
- `packages/shared/src/` - Shared types and converters

### API Modules

| Module | Path | Description |
|--------|------|-------------|
| Health | `/api/health` | Dependency health checks (DB, Redis, Meilisearch, storage) |
| Auth | `/api/auth` | Register, login, current user (`/api/auth/register`, `/api/auth/login`, `/api/auth/me`) |
| Cards | `/api/cards` | CRUD, public list/search, admin list (`/api/cards/admin/list`), publish/unpublish |
| Tags | `/api/tags` | List and create tags |
| Files | `/api/files` | Upload, download, card export (platform JSON, SillyTavern V2, TavernAI) |
| Admin | `/api/admin` | Bootstrap detection and admin creation |
| Passkey | `/api/passkey` | WebAuthn registration/login challenge and verify |
| Orders | `/api/orders` | Create orders (card or collection), user list, admin list (`/api/orders/admin/list`) |
| Payments | `/api/payments` | Stripe, YiPay, and epay webhook endpoints |
| Collections | `/api/collections` | CRUD, public list/search, admin list (`/api/collections/admin/list`), publish/unpublish, ZIP export |
| Audit | `/api/audit` | Audit log listing and per-action config |
| Users | `/api/users` | Admin user management (list, role update, delete) |
| Config | `/api/config` | System config key-value store |

### Web Routes

| Route | Page | Description |
|-------|------|-------------|
| `/cards` | `pages/cards/index` | Mobile-friendly marketplace: search, filter drawer, sort, grid/table, detail drawer, paid order creation |
| `/collections` | `pages/collections/index` | Public collection market: search, detail view, whole ZIP export, paid order creation |
| `/admin` | `pages/admin/index` | Mobile admin task flow: overview, card management, collection management, order lookup, config, audit |
| `/login` | `pages/login/index` | Mobile-safe auth page (JWT login + passkey), `layout: false` |
| `/register` | `pages/register/index` | Mobile-safe registration page, `layout: false` |

The web app uses a sticky top nav bar layout (`layouts/index.tsx`) with a horizontal menu (cards, collections) and auth buttons. On mobile (< 640px) the nav collapses into a hamburger drawer. The API client wrapper (`services/api.ts`) uses fetch with JWT from localStorage and returns `{ data, error }` for all calls.

## Run Modes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kkive/cardhub1](https://github.com/kkive/cardhub1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
