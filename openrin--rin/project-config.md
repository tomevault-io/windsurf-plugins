---
trigger: always_on
description: bun dev                    # Start both client and server
---

# AGENTS.md - Rin Project Guidelines

## Build Commands

```bash
# Development (using Rin CLI)
bun dev                    # Start both client and server
bun dev:client            # Client only (Vite + React)
bun dev:server            # Server only (Wrangler dev server on port 11498)
bun dev:cron              # Server with cron triggers enabled

# Building
bun run build             # Build all workspaces (turbo)
bun run check             # TypeScript type check (turbo)

# Database (using Rin CLI)
bun run db:generate       # Generate Drizzle migrations
bun run db:migrate        # Run local database migrations

# Formatting
bun run format:check      # Check formatting
bun run format:write      # Fix formatting

# Deployment (using Rin CLI)
bun run deploy            # Deploy both frontend (Pages) and backend (Workers)
bun run deploy:server     # Deploy backend only
 bun run deploy:client     # Deploy frontend only

# Release (using Rin CLI)
bun run release <version> # Create a new release (patch/minor/major/x.y.z)

# Testing
bun run test              # Run all tests (client + server)
bun run test:server       # Run server tests only
 bun run test:coverage     # Run tests with coverage report
```

## Rin CLI

The project uses a unified CLI tool located at `cli/bin/rin.ts`. Command implementations live under `cli/src/{commands,tasks,lib}`.

### CLI Commands
- `bun cli/bin/rin.ts dev [options]` - Start development server
- `bun cli/bin/rin.ts deploy [options]` - Deploy to Cloudflare
- `bun cli/bin/rin.ts db migrate` - Run database migrations
- `bun cli/bin/rin.ts release <version>` - Create a new release

## Architecture Status

This repository is a product monorepo, not a framework monorepo.

- `rin` should learn from `~/projects/rine` on module boundaries and type discipline.
- `rin` should NOT blindly copy `rine`'s `contracts/core/renderer/adapters/apps` layout.
- The current repository still centers around a concrete app: React frontend, Cloudflare Worker backend, shared API package, and local CLI tooling.
- If a change improves directory symmetry but does not improve dependency boundaries, it is usually the wrong refactor.

### Current Reality

- `client/` is the web application, not a reusable package.
- `server/` is the Worker application, not a generic framework core.
- `packages/api/` is the only stable shared package today.
- `packages/ui/` exists in the tree but is not yet a real workspace package; treat it as incomplete work until it has its own `package.json`, tsconfig, exports, and consumers.
- `cli/` is an engineering tool for this repo, not a runtime dependency of the product.

### Refactor Direction

When restructuring, prefer this target shape over a direct clone of `rine`:

```text
rin/
├── apps/
│   ├── web/            # current client app
│   └── worker/         # current server app
├── packages/
│   ├── api/            # shared request/response types and schemas
│   ├── ui/             # real reusable UI primitives and markdown-related UI
│   ├── web-core/       # app shell, providers, routing, layout composition
│   ├── server-core/    # app assembly, middleware, route registration, env typing
│   └── config/         # shared config keys, defaults, parsing, client-safe views
├── tools/
│   └── cli/            # current CLI moved only when boundaries are stable
```

This is a direction, not a mandate to rename everything immediately.

## Refactor Guardrails

All agents must follow these rules for architecture work:

1. Do not start by renaming `client/` to `apps/web` or `server/` to `apps/worker`.
2. Do not create empty framework-style packages just to mirror `rine`.
3. Do not introduce a `renderer` package unless the app actually gains SSR or shared rendering orchestration.
4. Do not split every server utility into `adapters/*` packages unless there are at least two credible implementations or an immediate runtime boundary that justifies it.
5. Do not move code across the repo solely for aesthetics if imports, tests, and ownership get worse.

### Required Refactor Order

For any substantial restructure, agents should work in this order:

1. Stabilize boundaries.
   - Create real workspace packages before migrating code into them.
   - Add missing package manifests, tsconfig files, and explicit exports.
   - Introduce a root TypeScript base config before broad moves.
2. Extract shared logic.
   - Move truly shared UI into `packages/ui`.
   - Move shared config models and parsing into a dedicated shared package.
   - Keep app-specific composition in the app layer.
3. Simplify application entrypoints.
   - Split frontend bootstrap, providers, routes, and layout composition.
   - Split backend app assembly, middleware registration, and runtime entry handling.
4. Rename directories only after the dependency graph is cleaner.

If a proposed refactor skips step 1 and jumps to step 4, it should be treated as suspect.

## Layering Rules

### Frontend

- `client/src/App.tsx` should trend toward composition only: providers, routes, and top-level layout wiring.
- Page initialization, session loading, config hydration, and permission gating should move into dedicated modules when touched.
- Reusable presentational components belong in `packages/ui` only when they are app-agnostic.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openRin/Rin](https://github.com/openRin/Rin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
