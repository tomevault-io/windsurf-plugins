---
trigger: always_on
description: Cypress is an open-source end-to-end and component testing framework for the modern web. This monorepo ships the Cypress desktop application and CLI (`cypress`), the JavaScript driver that runs tests in the browser, the Electron-based test runner, a suite of published npm packages (component testing adapters, webpack/vite dev-server integrations, plugins), and the internal tooling used to build and release all of it. Cypress is used by millions of developers to test web applications across Chrom
---

# Cypress Monorepo — AGENTS.md

## Overview

Cypress is an open-source end-to-end and component testing framework for the modern web. This monorepo ships the Cypress desktop application and CLI (`cypress`), the JavaScript driver that runs tests in the browser, the Electron-based test runner, a suite of published npm packages (component testing adapters, webpack/vite dev-server integrations, plugins), and the internal tooling used to build and release all of it. Cypress is used by millions of developers to test web applications across Chrome, Firefox, Edge, WebKit, and Electron.

## Workspaces

- **`cli/`** — The main `cypress` npm package (CLI entry point) and co-located component testing framework adapters (`@cypress/react`, `@cypress/vue`, `@cypress/angular`, `@cypress/svelte`, `@cypress/mount-utils`)
- **`packages/`** — Core internal packages: the test driver, Electron app, HTTP server, proxy, rewriter, launcher, frontend Vue app, launchpad, reporter, config, data-context, telemetry, types, errors, and more (33 packages total)
- **`npm/`** — Publicly published npm packages: bundler integrations, component testing adapters, plugins, and dev tooling (15 packages)
- **`tooling/`** — Internal build tooling: V8 snapshot creation, `packherd` dependency bundler, and `electron-mksnapshot` (3 packages)
- **`system-tests/`** — Full end-to-end system test suite run against a built Cypress binary
- **`scripts/`** — Internal build, release, and CI automation scripts

## Prerequisites

- **Node**: Use the node version specified in the `.node-version` file (check with `node -v`; run `nvm use` to manage versions)
- **Package manager**: Yarn 1 (`yarn@1.22.22`) — do not use npm or pnpm
- **Lerna**: Orchestrated via root `package.json` scripts; installed as a dev dependency
- **Electron** (for binary builds): handled automatically by `@packages/electron` during build

## Common Commands

### Setup & Development

```bash
# Install all dependencies (runs post-install hooks automatically)
yarn

# Start Cypress in dev mode (watch, rebuilds on change)
yarn dev

# Open the Cypress GUI in dev + global mode
yarn start

```

### Testing

```bash
# Run tests scoped to a single package (preferred over bare yarn test)
yarn test --scope @packages/server

# Target a specific vitest spec file (packages that use vitest)
yarn workspace @packages/config test -- <path-to-spec>

# Target a specific vitest spec by glob pattern
yarn workspace @packages/net-stubbing test -- "<glob-pattern>"

# Target a specific mocha spec file (packages that use mocha)
yarn workspace @packages/server test-unit -- <path-to-spec>

# Filter mocha tests by name pattern
yarn workspace @packages/server test-unit -- --grep "<pattern>"

# Run system tests (full binary-level E2E)
yarn test-system

# Run Cypress headlessly against a specific spec (dev mode)
yarn cypress:run -- --spec "path/to/spec.cy.ts"

# Run Cypress component tests against a specific spec (dev mode)
yarn cypress:run:ct -- --spec "path/to/spec.cy.ts"
```

### Type Checking

```bash
# Type-check all TypeScript across the monorepo
yarn type-check

# Lerna-only type check pass
yarn check-ts
```

### Linting & Formatting

```bash
# Lint all packages (no bail, concurrency 2)
yarn lint

# Lint and auto-fix specific scopes
yarn lint:fix
```

> **Note**: This project does **not** use Prettier. All formatting is enforced via ESLint.

### Build

```bash
# Full monorepo build
yarn build

# Build V8 snapshot (dev)
yarn build-v8-snapshot-dev

# Build V8 snapshot (prod)
yarn build-v8-snapshot-prod

# Clean all build artifacts
yarn clean

# Clean and reinstall (nuclear)
yarn clean-deps && yarn
```

## Architecture

### CLI & Distribution

- **`cypress` (`cli/`)** — The `cypress` npm package users install. Entry point for `cypress open`, `cypress run`, `cypress install`, etc. Version: 15.x.

### Test Runner & Driver

- **`@packages/driver`** — The JavaScript test driver that executes user test code inside the browser. Implements Cypress commands, assertions, retries, and all `cy.*` APIs.
- **`@packages/runner`** — The webpack-bundled runner UI that hosts the AUT (application under test) iframe and driver communication layer.
- **`@packages/app`** — The Vue 3 frontend for the Cypress GUI / Launchpad. Main visual interface for the desktop app.
- **`@packages/launchpad`** — Project creation, onboarding, and test file scaffold UI.
- **`@packages/frontend-shared`** — Shared Vue components and design system tokens used by `app` and `launchpad`.
- **`@packages/reporter`** — The test results reporter UI component (pass/fail tree, log panel).

### Server & Network

- **`@packages/server`** — HTTP server responsible for serving test files, handling browser launching, socket communication, and orchestrating the test run.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cypress-io/cypress](https://github.com/cypress-io/cypress) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
