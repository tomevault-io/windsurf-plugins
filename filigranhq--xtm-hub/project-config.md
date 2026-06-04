---
trigger: always_on
description: - **No console.log** — Use `logApp` (backend) from `src/utils/app-logger.util.ts`. `console.warn` and `console.error` are only allowed in scripts or launch code not directly related to the app.
---

# XTM Hub - Coding Agent Instructions

## Critical Rules

- **No console.log** — Use `logApp` (backend) from `src/utils/app-logger.util.ts`. `console.warn` and `console.error` are only allowed in scripts or launch code not directly related to the app.
- Unused variables must be prefixed with `_` (e.g. `_unused`).

## Repository Overview

XTM Hub is the unified entry point for Filigran's ecosystem — a marketplace for cybersecurity resources, knowledge-sharing platform, and community engagement hub. It is a full-stack TypeScript monorepo.

- **Version**: 1.5.3
- **Architecture**: Yarn 4 workspaces monorepo
- **Runtime**: Node.js 24.13.1 (see `.nvmrc`)
- **Package Manager**: Yarn 4.12.0 via Corepack (`packageManager` field in root `package.json`)

### Applications

| Workspace          | Path                    | Stack | Dev Port |
|--------------------|-------------------------|---|---|
| `backend`          | `apps/backend`          | Express 5, Apollo Server, GraphQL, Knex, PostgreSQL, Elasticsearch, MinIO | 4002 |
| `frontend`         | `apps/frontend`     | Next.js 15 (App Router + Turbopack), React 19, Relay 20, TailwindCSS 4, `@filigran/ui` | 3002 |
| `e2e`              | `apps/e2e`              | Playwright | — |

## Setup

```bash
corepack enable          # REQUIRED — without this, yarn commands fail with version mismatch
yarn install             # from repo root, installs all workspaces
```

`corepack enable` must run before ANY yarn command. The global yarn (1.x) will NOT work.

## Development

### Local Infrastructure (Docker Compose)

```bash
docker compose -f xtm-hub-dev/docker-compose.yml up
```

Starts: PostgreSQL (5434), MinIO (9002), Elasticsearch (9204), Kibana (5603), PgAdmin (8888), Mailpit (8025/1025).

### Dev Servers

```bash
yarn dev:api             # starts backend on :4002
yarn dev:front           # starts frontend on :3002 (needs API running first)
```

## Build & Validation

### Backend (`apps/backend`)

```bash
yarn build               # esbuild compile + copy .graphql and migration .js files
yarn check-ts            # TypeScript type check (noEmit)
yarn lint                # ESLint
yarn lint:fix            # ESLint auto-fix
yarn test                # Vitest (sets VITEST_MODE=true)
yarn test:coverage       # with V8 coverage
```

### Frontend (`apps/frontend`)

```bash
yarn relay               # REQUIRED before build — runs relay-compiler + generate-enum script
yarn build               # relay + Next.js production build
yarn check-ts            # TypeScript type check
yarn lint                # next lint (ESLint)
yarn lint:fix            # auto-fix not available via next lint, use prettier
yarn test                # Vitest + jsdom
yarn test:coverage       # with V8 coverage
```

**`yarn relay` must run after any GraphQL schema change.** The `dev` script runs relay-compiler automatically via `concurrently`.

### E2E Tests (`apps/e2e`)

```bash
yarn test:e2e            # Playwright (requires frontend + backend running)
yarn test:e2e:ui         # Playwright UI mode
```

E2E runs with `workers: 1` (sequential), `retries: 2`, Chromium only. Base URL defaults to `http://localhost:3002`.

## GraphQL Pipeline

This is the most important data flow to understand:

1. **Schema definition**: `.graphql` files in `apps/backend/src/modules/**/` and `src/nodes/`
2. **Backend codegen**: `yarn generate:ts` in backend → runs `graphql-codegen` → produces `src/__generated__/resolvers-types.ts`
3. **Schema export**: When `NODE_ENV` is not production/staging/development, the API writes `schema.graphql` to `apps/frontend/schema.graphql`
4. **Relay compilation**: `yarn relay` in frontend → reads `schema.graphql` → generates TypeScript artifacts in `apps/frontend/__generated__/`
5. **Enum generation**: `yarn generate:enum` (part of `yarn relay`) → extracts enums from the schema into TypeScript

GraphQL resolvers are merged in `src/server/graphql-schema.ts`. Each module typically has: `*.graphql` (schema), `*.resolver.ts`, `*.service.ts`.

## Repository Structure

### Root

```
.nvmrc                  # Node 24.13.1
.yarnrc.yml             # Yarn 4 config: node-modules linker, scripts disabled, 3-day age gate
.rules                  # Project rules (no comments in code)
tsconfig.json           # Base TS config (extended by workspaces)
graphql.config.yml      # Points to apps/backed/**/*.graphql
codecov.yml             # Coverage reporting (informational only)
renovate.json           # Dependency automation
.husky/pre-commit       # Runs lint-staged in backend then frontend
chart/                  # Helm chart for Kubernetes deployment
xtm-hub-dev/            # Docker Compose files (dev + CI)
```

### Backend — `apps/backend/`

```
src/index.ts                    # Entry point — Express + Apollo Server + SSE setup
src/config.ts                   # Configuration via node-config library
src/crons.ts                    # Scheduled jobs (node-cron)
src/portal.const.ts             # Platform constants (UUIDs, roles, system user)
src/pub.ts                      # GraphQL PubSub for subscriptions
src/session-store-manager.ts    # Session store (PostgreSQL or memory)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FiligranHQ/xtm-hub](https://github.com/FiligranHQ/xtm-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
