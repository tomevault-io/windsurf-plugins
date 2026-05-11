---
trigger: always_on
description: Concise, enforceable standards for this app. Enough context to work effectively without bloating agent memory.
---

# Engineering Guidelines (AGENTS)

Concise, enforceable standards for this app. Enough context to work effectively without bloating agent memory.

## What This App Is
- Mission: Manage and analyze red‑team operations and defensive effectiveness.
- Core objects: Operations → Techniques → Outcomes, plus Taxonomy (tags, tools, log sources, crown jewels, threat actors) and MITRE data.
- Users and roles: ADMIN, OPERATOR, VIEWER. Groups are optional to restrict access per operation.
- Tech stack: Next.js 15 (App Router) + TypeScript, tRPC v11 + Zod, Prisma, NextAuth v5, Tailwind.
- Local development runs the Next.js dev server against a local PostgreSQL container; production environments use Docker with Postgres (user provides their own reverse proxy)

## Quick Commands
- `npm run db:up` — Start local Postgres (`deploy/docker/docker-compose.dev.yml`)
- `npm run init` — Apply migrations + seed baseline data
- `npm run dev --turbo` — Start dev server
- `npm run db:migrate -- --name <change>` — Create a new migration during development
- `npm run db:deploy` — Apply committed migrations to the current database
- `npm run seed:demo` — Populate optional demo taxonomy/operation data
- `npm run check` — Lint + type-check (must be clean)
- `npm run test` — Run tests
- `npm run build` — Production build

## Core Principles
- Security first: all app routes and tRPC procedures require authentication. No public endpoints other than login.
- Single source of truth: no duplicated APIs for the same data/metric.
- Keep API shapes stable during structural refactors.
- Predictable typing: no `any`. Use Zod to validate inputs and Prisma types for DB results.
- Prefer small, composable modules; target **300–700 LoC** per file.

## Target Structure & Imports
```
src/
  app/                         # Next.js app router
  features/                    # Domain UI + hooks (operations, analytics, settings, shared)
  components/ui/               # Shared UI primitives only
  lib/                         # Framework‑agnostic utilities and MITRE helpers
  server/
    api/routers/               # All tRPC routers (entity + analytics)
    services/                  # Shared DB/service logic
    auth/                      # NextAuth config/callbacks
    db/                        # DB bootstrap helpers
  test/                        # Vitest tests, factories, utilities
  types/                       # Global ambient types
```
- Do not create routers under `src/features/**`; keep them in `src/server/api/routers/**`.
- Keep analytics-only logic under `src/server/api/routers/analytics/**` (no CRUD).
- `src/lib/**` remains React-free.

### Path Aliases (`tsconfig.json`)
```
"paths": {
  "@/*": ["src/*"],
  "@features/*": ["src/features/*"],
  "@server/*": ["src/server/*"],
  "@lib/*": ["src/lib/*"],
  "@components/*": ["src/components/*"]
}
```

### Lint Boundaries
Use `eslint-plugin-boundaries` and `no-restricted-imports` to discourage cross‑feature leaks.

## Auth & Access
- Use `protectedProcedure`/`viewerProcedure`/`operatorProcedure`/`adminProcedure` (no public procedure).
- Use the shared access helpers from `src/server/api/access.ts` everywhere you need operation scoping:
  - `getAccessibleOperationFilter(ctx)` — list queries
  - `checkOperationAccess(ctx, operationId, action)` — view/modify checks
- Group-based rule: operations are either `EVERYONE` or `GROUPS_ONLY`. Non-admins must belong to at least one of the operation's `accessGroups` when visibility is `GROUPS_ONLY`; `EVERYONE` operations are visible to all authenticated roles.
- Redirect policy: Middleware gates all routes; unauthenticated API calls get 401 JSON, and pages redirect to `/auth/signin` (with `callbackUrl`).
- Layout gating (route group pattern):
  - Put all protected pages under `src/app/(protected-routes)/**` and add a server `layout.tsx` in that group that calls `auth()` and redirects to `/auth/signin` when missing. This prevents static prerender.
  - Do not duplicate auth checks in child layouts/pages under the group. Rely on the group layout for auth.
  - Keep `src/app/(protected-routes)/settings/layout.tsx` for the admin-only rule; it should only enforce `session.user.role === ADMIN` (assumes auth already passed).
  - Keep public auth at `src/app/(public-routes)/auth/signin/**`.
  - Demo mode login is optional; when enabled it should expose a single button for the initial admin on the sign-in page.
  - The homepage `/` is under the protected group and does not need page-level `auth()`.

## API Architecture
- Routers by concern:
  - Entity routers: CRUD and simple queries.
  - Analytics router: aggregations and metrics only (no CRUD). Keep analytics here; do not duplicate analytics in entity routers.
- Consistent shapes: list and get endpoints should return the same entity shape.
- Input validation: all endpoints validate with Zod; avoid optional/loose shapes when a stricter one is known.
- Error semantics: prefer `TRPCError({ code, message })`; do not leak internal details.
- Services layer: Routers validate/auth, then call small service functions for shared DB logic. Services throw `TRPCError` on validation failures and return Prisma-typed results. This avoids duplication across routers and import flows.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [initstring/RTAP](https://github.com/initstring/RTAP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
