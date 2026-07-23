---
trigger: always_on
description: This file provides guidance to AI coding agents working with this repository.
---

# Agent Guide

This file provides guidance to AI coding agents working with this repository.

## What is ThunderHub

ThunderHub is a Lightning Network node manager. It provides a web UI and GraphQL API for managing LND nodes — channels, payments, invoices, forwards, peers, chain transactions, and Boltz submarine swaps. It integrates with Amboss for node analytics and health monitoring.

## Commands

```bash
npm run build                                    # Build NestJS server + Vite client
npm run build:nest                               # Build server only
npm run build:client                             # Build client only
npm run start:dev                                # Dev: NestJS watch + Vite dev server (concurrent)
npm run lint                                     # ESLint with auto-fix
npm run lint:check                               # ESLint without fix (used in CI)
npm run test                                     # Jest (rootDir: src/, matches *.spec.ts)
npm run test -- --testPathPattern="channels"     # Run tests matching a pattern
npm run test:e2e                                 # E2E tests (config: test/jest-e2e.json)
npm run generate                                 # GraphQL codegen — requires server running at localhost:3000; not for CI
```

**Note:** Package manager is `npm`, not pnpm/yarn. Node version: see `.nvmrc`.

## Verifying changes

After making code changes, run these in order to confirm correctness:

```bash
npm run lint:check   # Check for lint errors (no auto-fix)
npm run test         # Run unit tests
npm run build        # Confirm the full build succeeds
```

Run a targeted test when touching a specific module:

```bash
npm run test -- --testPathPattern="<module-name>"
```

### Regenerating GraphQL types

When changes touch GraphQL resolvers, types, queries, or mutations, you **must** regenerate the typed client code:

1. Start the dev server: `npm run start:dev`
2. Once the server is listening on `localhost:3000`, run: `npm run generate`
3. Commit the updated `schema.gql` and `__generated__/*.generated.tsx` files alongside your changes.

Skipping this step will cause the client to use stale types that may not match the server schema.

## Architecture

Monorepo with two apps under `src/`:

- **`src/server/`** — NestJS backend (GraphQL API via Apollo, code-first schema)
- **`src/client/`** — React frontend (Vite, Apollo Client, React Router)

Each has its own `tsconfig.json`. The root tsconfig only includes `src/server`. The client tsconfig at `src/client/tsconfig.json` is strict mode with `@/*` aliased to `./src/*`.

### Server (`src/server/`)

**Entry:** `main.ts` bootstraps NestJS with Helmet, Winston logger, and optional `BASE_PATH` prefix.

**Root module:** `app.module.ts` wires up GraphQL (Apollo Driver, code-first), static file serving, JWT auth context, dataloaders, and scheduled tasks.

**Config:** `config/configuration.ts` is the single place all env vars are read. Returns a typed `ConfigType`. Uses `@nestjs/config` (global). Env files: `.env.local` overrides `.env`.

**Module layout under `modules/`:**

- **`api/`** — GraphQL resolvers organized by domain. Each subdomain (channels, invoices, wallet, boltz, amboss, etc.) has `*.module.ts`, `*.resolver.ts`, `*.types.ts`, and optionally `*.helpers.ts`.
- **`node/`** — LND abstraction layer. `NodeService` is the facade resolvers call; it resolves the account by user ID, then delegates to `LndService`. `LndService` wraps the `lightning` npm package directly.
- **`accounts/`** — In-memory account store. `AccountsService` implements `OnModuleInit`: at startup, reads SSO config and account config files, creates authenticated LND gRPC connections (`authenticatedLndGrpc`), and stores `EnrichedAccount` objects (account data + `lnd` handle) in a map keyed by account hash.
- **`security/`** — Three global guards registered as `APP_GUARD`: `GqlAuthGuard` (JWT via passport), `RolesGuard`, `GqlThrottlerGuard`. Key decorators: `@Public()` (skip auth), `@Roles(...)`, `@CurrentUser()` (extracts `UserId` from GQL context).
- **`dataloader/`** — Creates per-request `DataLoader` instances for batching Amboss API lookups (`nodesLoader`, `edgesLoader`). Injected into GraphQL context.
- **`fetch/`** — HTTP client with optional SOCKS proxy (Tor) support. `graphqlFetchWithProxy()` for external GraphQL APIs.
- **`sub/`** — LND event subscriptions (invoices, payments, forwards, channels, backups) using `async.auto()` with retry logic. Emits events to `SseService` for real-time client updates.
- **`sse/`** — Server-sent events endpoint for pushing LND events to the client.

### Request flow (server)

```
HTTP request → GqlAuthGuard (JWT validation via passport) → RolesGuard → ThrottlerGuard
  → Resolver receives @CurrentUser() with { id: accountHash }
  → Resolver calls NodeService.method(user.id, ...)
  → NodeService looks up EnrichedAccount by hash (in-memory map)
  → NodeService delegates to LndService.method(account, ...)
  → LndService calls lightning library with account.lnd handle
```

### Async error helpers (`utils/async.ts`)

- `to<T>(promise)` — Awaits and returns result, throws on error
- `toWithError<T>(promise)` — Returns `[data, undefined] | [undefined, error]` tuple (Go-style)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apotdevin/thunderhub](https://github.com/apotdevin/thunderhub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
