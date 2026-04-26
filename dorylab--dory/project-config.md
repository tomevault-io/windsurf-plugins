---
trigger: always_on
description: This repository is a Yarn workspace monorepo for Dory.
---

# AGENTS.md

## Overview

This repository is a Yarn workspace monorepo for Dory.

- `apps/web`: main product application
- `apps/admin`: admin application
- `apps/electron`: Electron wrapper for the web app
- `packages/auth-core`: shared auth package

Most product work happens in `apps/web`.

## Product Model

The product has two runtime forms:

- Desktop
- Web self-hosting

These are runtime variants of the same `apps/web` application, not two separate frontends.

- Runtime is determined by `DORY_RUNTIME` and `NEXT_PUBLIC_DORY_RUNTIME`
- Do not scatter direct runtime env checks in feature code
- Use `apps/web/lib/runtime/runtime.ts` as the runtime helper layer
- Direct runtime env reads are only allowed inside the runtime helper layer or boot/injection code such as Electron startup

## Core Source Areas

Treat these as the primary source of truth:

- `apps/web/app`
- `apps/web/components`
- `apps/web/lib`
- `apps/web/lib/database`
- `apps/web/lib/database/postgres/schemas`
- `apps/web/lib/database/postgres/impl`
- `apps/web/lib/connection/drivers`
- `apps/web/lib/utils/ensure-connection.ts`
- `apps/web/hooks`
- `apps/web/shared`
- `apps/admin/app`
- `apps/admin/lib`
- `apps/electron/main`
- `packages/auth-core/src`
- `tests/e2e`

Avoid editing generated or local artifact paths unless the task explicitly targets them:

- `**/.next/**`
- `**/node_modules/**`
- `apps/web/dist-scripts`
- `apps/web/release`
- `apps/web/.tmp`
- `apps/web/localdata`
- `apps/electron/dist`
- `apps/electron/dist-electron`
- `apps/electron/dist/mac-arm64`
- `playwright-report`
- `test-results`

## Commands

Run from the repository root unless workspace-local execution is more appropriate.

- `yarn`
- `yarn dev`
- `yarn admin:dev`
- `yarn electron:dev`
- `yarn build`
- `yarn admin:build`
- `yarn electron:build`
- `yarn lint`
- `yarn typecheck`
- `yarn format:check`
- `yarn format:write`
- `yarn test:e2e`

Useful workspace commands:

- `yarn workspace web run lint`
- `yarn workspace web run typecheck`
- `yarn workspace admin run lint`
- `yarn workspace admin run typecheck`

## Coding Rules

- Make minimal, targeted changes
- Preserve existing structure and naming
- Follow the repository formatter: 4 spaces, semicolons, single quotes, trailing commas
- Keep imports aligned with the existing Prettier import sorting rules
- Use strict TypeScript patterns; do not weaken types to force builds through
- In `apps/web`, prefer the existing `@/*` alias where helpful
- UI work must use the existing theme system and component styling conventions
- Do not create one-off visual systems, isolated color tokens, or theme logic that bypasses the current theme infrastructure
- Reuse the existing theme providers, theme tokens, and themed components in `apps/web/app/themes.css`, `apps/web/app/globals.css`, and `apps/web/components/*` when building UI
- If a new UI pattern needs theme support, extend the existing theme system instead of generating an unthemed or separately themed implementation

## Database Rules

Dory is a multi-database product. The app's own persistence layer lives in `apps/web/lib/database`.

- Application storage currently supports `pglite` and `postgres`
- Shared application schema definitions live in `apps/web/lib/database/postgres/schemas`
- Application persistence implementations live in `apps/web/lib/database/postgres/impl`
- Keep schema definition separate from query and persistence logic
- Extend the database layer instead of placing persistence logic in routes or unrelated modules
- If a route needs new persistence behavior, add it in the database layer first and call that abstraction from the route
- New route code must not import low-level DB primitives for app persistence, including `drizzle-orm`, raw schema tables, or direct DB clients
- Existing direct DB access in routes should be treated as legacy code to reduce over time, not as a pattern to copy

## Connection Rules

Multi-connection support lives under `apps/web/lib/connection/drivers`.

- All driver-specific connection implementations should live in this directory tree
- If a new database driver or connection capability is added, implement it in the connection layer, not in routes or pages
- API routes under `apps/web/app/api` should use `ensureConnection` from `apps/web/lib/utils/ensure-connection.ts` as the standard entrypoint
- Do not duplicate connection-id parsing, team scoping, or pool lookup in routes when `ensureConnection` covers the use case
- If `ensureConnection` is insufficient, extend the helper or connection layer first, then update the route

## Route Rules

Apply these rules to `apps/web/app/api/**/route.ts`.

Routes are thin transport layers only.

Routes may do:

- request parsing
- lightweight validation
- auth or team-context resolution
- `ensureConnection` for connection-backed APIs
- delegation to `lib/database`, `lib/connection`, or `lib/server`
- response shaping and status handling

Routes should not do:

- direct application persistence logic
- driver-specific connection implementation
- repeated runtime detection logic
- repeated connection resolution logic
- large business workflows that belong in `lib/*`

Preferred route flow:

1. Parse and validate input.
2. Resolve auth or team context.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dorylab/dory](https://github.com/dorylab/dory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
