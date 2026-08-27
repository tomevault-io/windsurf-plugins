---
trigger: always_on
description: This file provides guidance to AI assistants when working with code in this repository.
---

# Repository Guidelines

This file provides guidance to AI assistants when working with code in this repository.

## Project Structure & Module Organization

This is a **TypeScript monorepo** using pnpm workspaces with three packages:

```
packages/
├── web/           # Vite + TanStack Router SPA (React 19)
│   └── src/
│       ├── routes/        # TanStack Router file-based routes
│       ├── components/    # React components
│       ├── providers/     # React context providers
│       ├── hooks/         # Custom React hooks
│       └── api/           # API client functions
├── api/           # Hono API server (Node.js)
│   └── src/
│       ├── api/           # API routes
│       ├── ai-providers/  # AI provider integrations (40+)
│       ├── connectors/    # Database connectors
│       └── middlewares/   # Hono middlewares
└── shared/        # Shared types, Zod schemas, utilities
    └── src/
        ├── types/         # TypeScript types
        └── utils/         # Shared utilities
```

Other directories:
- **`tests/`**: Vitest suites mirroring packages (client, server, shared)
- **`examples/`**: Example implementations
- **`supabase/`**: Local dev DB config, migrations, `seed.sql`
- **`docker/`**: Docker configuration files

**Key path aliases**:
- `@web/*` - Web package (`packages/web/src/*`)
- `@api/*` or `@server/*` - API package (`packages/api/src/*`)
- `@shared/*` - Shared package (`packages/shared/src/*`)

## Essential Commands

**You must run these commands after modifying any file to ensure code quality:**
```bash
pnpm typecheck  # TypeScript type checking (uses Turborepo) - REQUIRED
pnpm check      # Biome linting and formatting - REQUIRED
pnpm check:fix  # Auto-fix linting and formatting issues
```

### Development Commands
```bash
# Installation and setup
pnpm install

# Database
supabase start  # Start local Supabase database
supabase stop   # Stop local database

# Development server (runs both web and API via Turborepo)
pnpm dev        # Start all dev servers in parallel
pnpm dev:web    # Start only web dev server (Vite on port 3000)
pnpm dev:api    # Start only API dev server (Hono on port 8787)

# Testing
pnpm test                      # Run all tests (excludes in-depth integration tests)
pnpm test path/to/test.ts      # Run specific test file
pnpm test:watch                # Run tests in watch mode

# In-depth integration tests (slower, more comprehensive)
INCLUDE_IN_DEPTH=true pnpm test

# Runtime checks (also run in CI; both are slow, so they are not part of `pnpm test`)
pnpm verify:worker     # Bundle and boot the API on workerd
pnpm verify:container  # Build the all-in-one image and smoke test it (needs Docker)

# Build (uses Turborepo with caching)
pnpm build      # Build all packages
pnpm build:web  # Build only web package
pnpm build:api  # Build only API package

# Code quality
pnpm lint       # Run linter
pnpm format     # Check formatting
pnpm format:fix # Auto-fix formatting

# API testing (all requests go through port 3000, proxied to API)
curl "http://localhost:3000/v1/endpoint" -H "Authorization: Bearer super-agents"
```

## Architecture

### Web Application (packages/web)
- **Framework**: Vite + TanStack Router (SPA mode)
- **Routing**: File-based routing in `src/routes/`
  - `_main.tsx` - Layout wrapper with sidebar
  - `$paramName` - Dynamic route parameters
  - `.index.tsx` - Index routes for parent paths
- **Auto-generated**: `routeTree.gen.ts` (do not edit, in .gitignore)

### API Server (packages/api)
- **Framework**: Hono web framework
- **Entry**: `src/server.ts` (Node.js) or `src/index.ts` (Cloudflare Workers)
- **Routes**: `src/api/v1/`

#### The API runs on two runtimes

`pnpm dev:api` runs `wrangler dev`, so **workerd is the development runtime**,
while the Docker image runs the same code on Node through `src/server.ts`.
Anything reachable from `src/index.ts` has to work on both. In practice:

- **No module-scope I/O, timers, or randomness.** Workers reject these outright
  — `utils/sse-event-manager.ts` starts its ping interval on first use for this
  reason. Prefer a first-request flag over doing work at import time.
- **Node-only packages have to stay off the Workers path.** A driver with native
  bindings fails to bundle. Where a package ships a Workers build (`/web` entry
  or a `workerd` export condition), use it.
- **`node:` builtins are the quiet case.** Wrangler's unenv layer substitutes a
  stub, so the import resolves and the Worker boots — it throws only when the
  stub is called. Neither CI check catches this; review does.

`pnpm verify:worker` bundles and boots the Worker, and runs in CI.

### Request Flow
```
Development:  Browser (:3000) → Vite proxy → API (:8787)
Production:   Browser (:3000) → Hono (:3000)
```

In development, Vite proxies `/v1/*` requests to the API server on port 8787.

In production there is no proxy: a single Hono process serves both. Because the
API carries a `/v1` base path, `/v1/*` reaches the API routes and every other
path falls through to the dashboard's static build. See `packages/api/src/server.ts`.

## API Structure (Hono-based)

Key API endpoints:
- `/v1/chat/completions` - OpenAI-compatible chat API
- `/v1/super-agents/agents` - Agent management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jorge-menjivar/super-agents](https://github.com/jorge-menjivar/super-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
