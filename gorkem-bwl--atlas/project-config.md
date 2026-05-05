---
trigger: always_on
description: Atlas is an all-in-one business platform with modular app architecture. Each app is self-contained in its own directory and registers via manifests.
---

# Atlas — Project Documentation

## Overview

Atlas is an all-in-one business platform with modular app architecture. Each app is self-contained in its own directory and registers via manifests.

**Stack:** React + TypeScript + Vite (client), Express + PostgreSQL + Drizzle ORM (server), shared types package.

**Product name:** Atlas (NOT AtlasMail). No email functionality exists.

---

## Documentation Index

Detailed documentation lives in `/docs/`. Read the relevant doc before building or modifying a feature.

| Document | What it covers | When to read |
|----------|---------------|--------------|
| **Live OpenAPI spec:** `/api/v1/openapi.json` + Scalar UI at `/api/v1/reference` (generated from `packages/server/src/openapi/paths/`). The old `docs/api-reference.md` is deprecated. | Every API endpoint — method, path, auth, request/response shapes | Building client hooks, testing endpoints, debugging API calls |
| [Database Schema](docs/database-schema.md) | All tables, columns, types, constraints, FK relationships, indexes | Adding tables, writing migrations, building queries |
| [App Architecture](docs/app-architecture.md) | App registry pattern, per-app features/routes/tables, adding new apps | Building a new app, understanding how apps register |
| [Design System](docs/design-system.md) | CSS variables, component library (31 components), layout patterns, i18n | Building UI, creating components, styling, translations |
| [Infrastructure](docs/infrastructure.md) | Docker, deployment, CI/CD, CLI, env vars, SSL, backups, monitoring | Deploying, configuring, troubleshooting production |
| [Architecture for agents](docs/architecture-for-agents.md) | Registry patterns, data flow, auth layers, UI primitives lookup, debugging recipes, hard rules | Onboarding to the codebase; before touching a new area |

---

## Monorepo Structure

```
packages/
  client/     — React frontend (port 5180)
  server/     — Express API (port 3001)
  shared/     — Shared TypeScript types
```

---

## App Architecture

Every app follows the same self-contained structure:

### Client (`packages/client/src/apps/{name}/`)
```
manifest.ts          — App metadata, routes, settings panels, sidebar config
page.tsx             — Main page component
components/          — App-specific components
hooks.ts             — Data fetching hooks (React Query)
settings-store.ts    — App settings (Zustand + server persistence)
```

### Server (`packages/server/src/apps/{name}/`)
```
manifest.ts          — App metadata, Express router, table list
routes.ts            — Express route definitions
controller.ts        — Request handlers
service.ts           — Business logic + database queries
```

### Current Apps

| App | ID | Color | Icon | Sidebar Order | Route |
|-----|----|-------|------|---------------|-------|
| CRM | crm | #f97316 | CrmIcon (brand) | 10 | /crm |
| HRM | hr | #10b981 | HrmIcon (brand) | 20 | /hr |
| Projects | projects | #0ea5e9 | ProjectsIcon (brand) | 25 | /projects |
| Calendar | calendar | #f97316 | CalendarIcon (brand) | 27 | /calendar |
| Sign | sign | #8b5cf6 | SignIcon (brand, hand-authored) | 30 | /sign-app |
| Invoices | invoices | #0ea5e9 | InvoicesIcon (brand, hand-authored) | 35 | /invoices |
| Drive | drive | #64748b | DriveIcon (brand) | 40 | /drive, /drive/folder/:id |
| Tasks | tasks | #6366f1 | TasksIcon (brand, full-bleed) | 60 | /tasks |
| Write | docs | #c4856c | WriteIcon (brand, full-bleed) | 70 | /docs, /docs/:id |
| Draw | draw | #e06c9f | DrawIcon (brand, full-bleed) | 80 | /draw, /draw/:id |
| System | system | #6b7280 | SystemIcon (brand) | 90 | /system |

> **Note:** CRM, HRM, Projects, Calendar, Drive, and Draw use custom multicolor brand SVGs (defined in `packages/client/src/components/icons/app-icons.tsx`) instead of lucide icons in the dockbar. Most render on a small white/light card via `BRAND_ICON_BACKGROUNDS` in `sidebar.tsx` and `home.tsx`. Draw is **full-bleed** — its SVG artwork is itself a backdrop and fills the dock card edge-to-edge (controlled by `FULL_BLEED_BRAND_ICONS` in `app-icons.tsx`). All other apps still use lucide. Calendar is **client-only** — there is no `packages/server/src/apps/calendar/`.

---

## Adding a New App

### 1. Shared types
Create `packages/shared/src/types/{name}.ts` with interfaces.
Add `export * from './{name}'` to `packages/shared/src/types/index.ts`.

### 2. Database
Add tables to `packages/server/src/db/schema.ts`.
Run `cd packages/server && npm run db:push` to sync the schema to Postgres.

### 3. Server app
Create directory `packages/server/src/apps/{name}/` with:
- `service.ts` — CRUD functions (import db, schema, drizzle-orm)
- `controller.ts` — Express handlers (extract auth from `req.auth!`)
- `routes.ts` — Express router (import authMiddleware)
- `manifest.ts` — ServerAppManifest

Register in `packages/server/src/apps/index.ts`:
```typescript
import { myServerManifest } from './{name}/manifest';
serverAppRegistry.register(myServerManifest);
```

### 4. Client app
Create directory `packages/client/src/apps/{name}/` with:
- `hooks.ts` — React Query hooks
- `page.tsx` — Page component using AppSidebar
- `components/` — App-specific components

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gorkem-bwl/atlas](https://github.com/gorkem-bwl/atlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
