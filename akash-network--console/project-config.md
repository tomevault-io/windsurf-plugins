---
trigger: always_on
description: This file aggregates all RFC (Request for Comments) contribution guidelines that have landed.
---

# Contribution Guidelines

This file aggregates all RFC (Request for Comments) contribution guidelines that have landed.

## General Project Description

This is a **monorepo** project using **npm workspaces**. All projects use **strict TypeScript** configuration (`"strict": true` in all `tsconfig.json` files), ensuring type safety across the entire codebase.

### Project Structure

The monorepo contains multiple applications and shared packages:

**Applications:**
- `apps/api` - Hono + OpenAPI REST API with tsyringe DI
- `apps/deploy-web` - Next.js frontend for deploying on Akash Network
- `apps/stats-web` - Next.js frontend for network statistics
- `apps/indexer` - Node.js blockchain indexer (Express, no DI)
- `apps/notifications` - NestJS service for handling notifications (NestJS built-in DI)
- `apps/log-collector` - Standalone log collector (vanilla Node.js + tsyringe)
- `apps/provider-proxy` - Hono service for proxying requests to providers (manual factory DI)
- `apps/provider-console` - Provider console application
- `apps/tx-signer` - Internal transaction signing service (Hono + tsyringe)

**Shared Packages:**
- `packages/database` - Database shared package (Drizzle ORM schemas)
- `packages/dev-config` - Shared ESLint, TypeScript, and Prettier configs
- `packages/docker` - Docker build utilities
- `packages/env-loader` - Environment variable loading (uses `@dotenvx/dotenvx`)
- `packages/http-sdk` - HTTP client SDK
- `packages/instrumentation` - OpenTelemetry instrumentation setup
- `packages/logging` - Logging utilities (Pino-based `LoggerService`)
- `packages/net` - Blockchain Network utilities
- `packages/network-store` - Network state management
- `packages/react-query-proxy` - React Query wrapper utilities
- `packages/react-query-sdk` - React Query SDK for console services
- `packages/releaser` - Release management utilities
- `packages/ui` - Shared UI components

### Tech Stack

- **Node.js**: >= 24.14.1 (enforced via Volta)
- **Package Manager**: npm 11.11.0
- **Framework**: Next.js 14.x, Hono, Nest.js
- **Database**: PostgreSQL with Drizzle ORM and legacy sequelize
- **Monorepo**: npm workspaces
- **Testing**: Vitest (primary), Jest (legacy in indexer & provider-console), Playwright (e2e), React Testing Library
- **Styling**: Material-UI, Emotion, Tailwind CSS
- **State**: React Query, Jotai
- **Blockchain**: Cosmos, Akash network

### Building the Project

#### Using Docker Compose (Recommended for Development)

Build production images:
```bash
npm run dc:build
```

Run services in development mode:
```bash
npm run dc:up:dev
```

Run specific service with dependencies:
```bash
npm run dc:up:dev -- deploy-web
```

Stop all services:
```bash
npm run dc:down
```

#### Building Individual Applications

Each application has its own build script. Navigate to the app directory and run:
```bash
cd apps/<app-name>
npm run build
```

### Running Tests

IMPORTANT: Before pushing, ALWAYS run in the affected app:
1. `npm test` (runs unit, integration, and functional tests)
2. `npm run lint -- --quiet`
3. `npx tsc --noEmit`

#### Unit Tests

All applications and packages use **Vitest** for unit testing.

**In applications:**
```bash
# Run all unit tests
cd apps/<app-name>
npm run test:unit

# Run with watch mode
npm run test:unit -- --watch
```

**In packages:**
```bash
# Run tests for all packages
npm run test -w ./packages

# Run tests for a specific package
cd packages/<package-name>
npm test
```

#### Integration Tests

Integration tests run against real infrastructure (database, etc.) and are currently used in `apps/api`.

- **Naming**: `*.integration.ts` (colocated next to source files, like unit tests)
- **Running**:
  ```bash
  cd apps/api
  npm run test:integration
  ```
- **Setup**: Requires Docker services running via `npm run test:ci-setup`
- **Vitest project**: Configured as a separate `integration` project in `vitest.config.ts` with its own setup files and extended timeouts (60s test, 30s hook)

#### Functional Tests

Functional tests are available in `apps/api`, `apps/notifications`, `apps/provider-proxy`:

```bash
cd apps/<app-name>
npm run test:functional

# With watch mode
npm run test:functional -- --watch
```

**CI Setup/Teardown for Functional Tests:**

Some services require Docker services to be running before functional tests can execute. These services provide `test:ci-setup` and `test:ci-teardown` scripts:

- **`apps/api`**:
  ```bash
  cd apps/api
  npm run test:ci-setup    # Start required Docker services
  npm run test:functional # Run functional tests
  npm run test:ci-teardown # Stop Docker services
  ```

- **`apps/notifications`**:
  ```bash
  cd apps/notifications
  npm run test:ci-setup     # Start database service
  npm run test:functional  # Run functional tests
  npm run test:ci-teardown # Stop Docker services
  ```

These scripts use Docker Compose (`dc`) to manage test dependencies. The setup script starts services in detached mode (`-d`), and teardown stops all services.

#### End-to-End (E2E) Tests

E2E tests are available in `apps/api`, `apps/deploy-web`, `apps/provider-proxy`:

```bash
cd apps/deploy-web
npm run test:e2e
```

E2E tests in deploy-web require:
- Playwright browsers installed: `npx playwright install`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [akash-network/console](https://github.com/akash-network/console) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
