---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Switch-to.eu is a platform helping users switch from non-EU digital services to EU alternatives. It provides migration guides, EU alternative listings, and a domain analyzer tool.

## Commands

```bash
pnpm install          # Install all dependencies
pnpm dev              # Dev server for all apps (Turborepo, persistent)
pnpm build            # Build all apps/packages (respects dependency graph)
pnpm lint             # Lint all apps/packages

# Per-app (run from app directory or use --filter)
pnpm --filter website dev
pnpm --filter website build
pnpm --filter keepfocus dev
pnpm --filter @switch-to-eu/plotty dev
```

```bash
# E2E smoke tests (website app)
pnpm --filter website test:e2e      # Run all Playwright smoke tests
pnpm --filter website test:e2e:ui   # Open Playwright UI for debugging
```

## Testing

Every app should have **two layers of tests**:

### 1. Playwright E2E Smoke Tests

Verify all pages render without errors in a production build. Each app has its own `playwright.config.ts` and `e2e/smoke.spec.ts`.

```bash
pnpm --filter website test:e2e           # website (port 3000)
pnpm --filter @switch-to-eu/plotty test:e2e   # plotty (port 3042)
pnpm --filter @switch-to-eu/listy test:e2e    # listy (port 5014)
pnpm --filter @switch-to-eu/privnote test:e2e # privnote (port 5016)
```

**Pattern** — every smoke test file follows this structure:
- Helper `expectPageOk(page, urlPath)` that checks status 200, no error overlay, body visible
- Loop over `locales = ["en", "nl"]` and test each page in both locales
- Config: `webServer.command` builds + serves the production app on the app's port

### 2. Vitest Unit Tests

```bash
pnpm --filter @switch-to-eu/keepfocus test    # keepfocus (component + timer tests)
pnpm --filter @switch-to-eu/plotty test       # plotty (tRPC router tests)
pnpm --filter @switch-to-eu/listy test        # listy (tRPC router tests)
pnpm --filter @switch-to-eu/privnote test    # privnote (tRPC router tests)
```

**tRPC router tests** (plotty, listy, privnote) use a shared in-memory Redis mock at `@switch-to-eu/db/mock-redis`. Pattern:

1. Import `MockRedis` from `@switch-to-eu/db/mock-redis`
2. Mock the redis module: `vi.mock("@switch-to-eu/db/redis", () => ({ getRedis: async () => mockRedis, getRedisSubscriber: async () => mockRedis }))`
3. Import the app's `createCaller` **after** mocks are set up (top-level await)
4. Create a caller with `createCaller({ redis: mockRedis as never, headers: new Headers() })`
5. Use `mockRedis._clear()` in `beforeEach` to reset state between tests
6. Seed data directly via `mockRedis.hSet()` / `mockRedis.sAdd()` for read/update/delete tests

Each app's vitest config lives at `apps/{app}/vitest.config.ts` with `environment: "node"` and path aliases matching `tsconfig.json`.

### 3. Testcontainers Integration Tests

For testing real Redis behavior (TTL, key deletion, hGetAll), use Testcontainers to spin up a real Redis container in Docker.

```bash
pnpm --filter @switch-to-eu/listy test:integration     # listy (requires Docker)
pnpm --filter @switch-to-eu/privnote test:integration  # privnote (requires Docker)
```

**Shared helper** at `@switch-to-eu/db/test-redis` provides `setupRedisContainer()`, `teardownRedisContainer()`, and `getTestRedisUrl()`. Pattern:

1. `beforeAll`: call `setupRedisContainer()` (sets `process.env.REDIS_URL`), then dynamically import the router module
2. Create a separate `redis` client via `createClient({ url: getTestRedisUrl() })` for direct assertions
3. `afterEach`: `redis.flushAll()` to reset between tests
4. `afterAll`: quit the client and call `teardownRedisContainer()`

Integration tests live in `__tests__/**/*.integration.test.ts` and are excluded from the default `vitest.config.ts`. A separate `vitest.integration.config.ts` includes only `*.integration.test.ts` with longer timeouts (`testTimeout: 30s`, `hookTimeout: 60s`).

**When to use each**: Use MockRedis for fast unit tests (CI, TDD). Use Testcontainers for verifying real Redis behavior (TTL, key expiration, concurrent access). Both should be maintained side by side.

## Monorepo Architecture

**Package manager:** pnpm with workspaces. **Build orchestration:** Turborepo.

```
apps/
  website/        # Main switch-to.eu site (Next.js 16, App Router)
  keepfocus/      # Pomodoro timer app (Next.js 16, App Router)
  plotty/         # Poll/voting app (Next.js 16, App Router, tRPC + Redis)
  listy/          # Shared lists app (Next.js 16, App Router, tRPC + Redis)
  privnote/       # Self-destructing notes app (Next.js 16, App Router, tRPC + Redis, E2E encryption)
packages/
  ui/             # Atomic UI components (shadcn/ui new-york style, Radix primitives)
  i18n/           # Shared i18n layer (next-intl v4)
  blocks/         # Page-level shared components (Header, Footer, LanguageSelector)
  db/             # Shared database utilities (Redis client, crypto, admin tokens, expiration, mock-redis for tests, test-redis for Testcontainers)
  trpc/           # Shared tRPC v11 infrastructure (used by plotty, listy, privnote — NOT by website or keepfocus)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [switch-to-eu/switch-to.eu](https://github.com/switch-to-eu/switch-to.eu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
