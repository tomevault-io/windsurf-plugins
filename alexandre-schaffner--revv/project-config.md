---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## What This Is

**Revv** is an AI-powered code review desktop application. It's a Tauri v2 desktop app with a SvelteKit frontend and a local Bun/Elysia API server that syncs GitHub pull requests and enables AI-assisted review workflows.

Stack: Bun + TypeScript monorepo (Turborepo), Svelte 5, Elysia, Drizzle ORM on SQLite, Tauri v2 (Rust).

## Commands

```bash
# Setup
bun install              # Install all workspace deps
cp .env.example .env     # Then fill in GITHUB_CLIENT_ID and GITHUB_CLIENT_SECRET

# Development
make dev                 # All 3 services (web @ 5173, server @ 45678, Tauri desktop)
make dev-web             # SvelteKit only
make dev-server          # Elysia API only

# Quality
make typecheck           # tsc across all packages
make lint                # Linters across all packages

# Build & Distribution
make build               # Build all packages
make dist                # Build platform installer (dmg/msi/deb)

# Database
cd apps/server && bunx drizzle-kit generate   # Generate migration from schema changes

# Cleanup
make clean               # Remove build artifacts
make reset-db            # Delete SQLite database (apps/server/revv.db)
```

## Architecture

### Monorepo Layout

- `apps/web` — SvelteKit frontend (served by Tauri, also accessible at `localhost:5173` in dev)
- `apps/server` — Elysia HTTP + SSE server (port 45678)
- `apps/desktop` — Tauri v2 shell; minimal Rust, just window + plugin setup
- `packages/shared` — Shared types, constants (`API_PORT`, `APP_NAME`), and SSE event message schemas

`packages/shared` is the source of truth for cross-app types. Import from `@revv/shared`.

### Server (`apps/server`)

- **Effect system** throughout: services use `Effect.gen`, `Context.Tag`, and `Layer` for DI and structured error handling. Don't bypass Effect when modifying services.
- **Services**: `GitHubService`, `RepositoryService`, `PullRequestService`, `PollScheduler`, `Broadcaster`, `Settings`, `TokenProvider`
- **Auth**: `better-auth` with GitHub OAuth. Bearer token strategy. OAuth callback URL: `http://localhost:45678/api/auth/callback/github`
- **Database**: Drizzle ORM on SQLite (`revv.db`). Schema in `src/db/schema.ts`. Migrations live in `src/db/migrations` and are auto-applied on startup via `drizzle-orm/bun-sqlite/migrator`. Generate new ones with `bunx drizzle-kit generate` (run from `apps/server`).
- **Realtime (SSE)**: Clients open a single `GET /api/events?token=…` stream. Server broadcasts `prs:updated`, `repos:updated`, etc. via `Broadcaster` (account-scoped fan-out). Inbound commands use REST endpoints.

#### Database migrations

The project uses **Drizzle's code-first migration workflow**.

**How it works:**
1. **Schema is the source of truth.** All tables, indexes, and relations are defined in TypeScript under `apps/server/src/db/schema/`.
2. **Generate a migration** after changing the schema by running `cd apps/server && bunx drizzle-kit generate`. This compares the current schema against the database state and writes a new `.sql` file + a `meta/` snapshot into `src/db/migrations/`.
3. **Migrations are applied automatically on startup.** `src/db/index.ts` calls `migrate()` from `drizzle-orm/bun-sqlite/migrator` using the `src/db/migrations` folder. You do **not** need to run `drizzle-kit migrate` or `drizzle-kit push` manually.
4. **Migration state is tracked** in a `__drizzle_migrations` table inside `revv.db`. Already-applied migrations are skipped on subsequent starts.

**Expected agent workflow when schema changes are needed:**
1. Edit the Drizzle schema files in `apps/server/src/db/schema/`.
2. Generate the migration: `cd apps/server && bunx drizzle-kit generate`.
3. Review the generated `.sql` file in `apps/server/src/db/migrations/` for correctness.
4. Restart the server (`make dev-server` or `make dev`). The startup routine will apply the new migration automatically.
5. If the generated migration is wrong (e.g., destructive defaults, wrong column types), **edit the generated `.sql` file before restarting** rather than trying to fix it after it has been applied.

**Do not** run `drizzle-kit push` against the local SQLite file — the app relies on the migration-file workflow for reproducible schema evolution.

### Web (`apps/web`)

- **Svelte 5 runes** (`$state`, `$derived`, `$effect`) — not Svelte 4 stores/writables.
- **Stores** (in `src/lib/stores/`): `auth.svelte.ts`, `prs.svelte.ts`, `events.svelte.ts`, `settings.svelte.ts`. These expose getter/setter functions, not subscribables.
- **API client**: Eden (Elysia type-safe client) — import from `@revv/server` types.
- **Deep-link handling**: OAuth callback comes in via `revv://auth/callback?token=…` scheme (Tauri) or polling `/api/auth/pending-token` (browser dev mode).
- **Component library**: shadcn-svelte + Tailwind CSS v4.

### Desktop (`apps/desktop`)

- Tauri v2. Frontend served from `../web/build`. Dev URL: `http://localhost:5173`.
- Plugins: `tauri-plugin-deep-link` (handles `revv://` scheme), `tauri-plugin-opener`.
- CSP restricts API calls to `localhost:45678`.

## TypeScript Config


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexandre-schaffner/revv](https://github.com/alexandre-schaffner/revv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
