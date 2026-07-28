---
trigger: always_on
description: **Interledger TestNet** is an open-source, full-stack Node.js/TypeScript monorepo demonstrating Interledger Protocol integration. It consists of:
---

# GitHub Copilot Instructions — Interledger TestNet

## Project Summary

**Interledger TestNet** is an open-source, full-stack Node.js/TypeScript monorepo demonstrating Interledger Protocol integration. It consists of:

- **Wallet Backend** (NestJS) — Account management, KYC, payment rails
- **Wallet Frontend** (Next.js) — User-facing web UI for accounts and transactions
- **Boutique Backend** (Express) — E-commerce demo server
- **Boutique Frontend** (Vite) — E-commerce storefront
- **Shared Packages** — Common backend & frontend utilities
- **E2E Tests** (`e2e/`) — Playwright + playwright-bdd (Gherkin) end-to-end tests against the local environment
- **Helm Charts** (`helm/`) — Kubernetes charts for `boutique` and `testnet-wallet`, published to the `charts` branch via GitHub Pages

**Size**: ~100K lines of TypeScript + Node.js; ~80 test files; ~10 npm packages
**Purpose**: Reference implementation for Account Servicing Entities integrating with Interledger Protocol and Rafiki
**Key Integrations**: Rafiki (ILP), MockGatehub (sandbox KYC/fiat), Stripe, GateHub, Kratos (identity)

---

## Prerequisites

### Required Environment

- **Node.js 24 LTS** (enforced by `package.json` engines field)
- **pnpm 9.x** (managed via Corepack; CI uses `pnpm/action-setup@v2`)
- **Docker** and **Docker Compose** (for local services: Postgres, Redis, Traefik, Kratos, Rafiki, MockGatehub)
- **Git**

### Setup Steps (First Time Only)

```bash
# 1. Switch to Node 24 (assumes nvm installed)
nvm install 24
nvm use 24

# 2. Enable Corepack (pnpm package manager)
corepack enable

# 3. Clone and navigate
cd /path/to/testnet

# 4. Install all dependencies
pnpm install --frozen-lockfile

# 5. (Optional) Local development setup (interactive, requires Docker, sudo)
pnpm local:setup
```

**Critical**: Always run `pnpm install --frozen-lockfile` after pulling; never use `npm install` or `yarn install`.

---

## Build & Validation Commands

### Core Commands (Work Immediately After Setup)

| Command         | Purpose                                      | Time  | Notes                                                                                |
| --------------- | -------------------------------------------- | ----- | ------------------------------------------------------------------------------------ |
| `pnpm checks`   | ESLint (--max-warnings=0) + Prettier check   | ~3s   | **Always run before PR** — catches formatting and linting issues                     |
| `pnpm test`     | Jest unit tests (wallet + boutique backends) | ~80s  | Runs `jest --passWithNoTests --maxWorkers` per package; uses experimental VM modules |
| `pnpm format`   | Auto-fix ESLint + Prettier                   | ~5s   | Mutates files in place; safe to run                                                  |
| `pnpm build`    | Compile all packages to `dist/` and `.next/` | ~30s  | Requires correct Node version; builds dependencies first                             |
| `pnpm e2e:test` | Playwright e2e tests (headless)              | ~2min | Requires full local stack running; see E2E Tests section below                       |

### Per-Package Commands

```bash
# Wallet backend
pnpm wallet:backend build     # Compile wallet backend (NestJS)
pnpm wallet:backend test      # Unit tests (Jest)
pnpm wallet:backend dev       # Watch mode (Requires local services up)

# Wallet frontend
pnpm wallet:frontend build    # Next.js production build
pnpm wallet:frontend dev      # Dev server (Requires backend services)

# Boutique backend
pnpm boutique:backend build    # Express app builds via TypeScript
pnpm boutique:backend test     # Jest unit tests
pnpm boutique:backend dev      # Watch mode

# Boutique frontend
pnpm boutique:frontend build   # Vite production build
pnpm boutique:frontend dev     # Dev server
```

### Repository Maintenance

```bash
pnpm clean             # Remove all node_modules/ and dist/.next/
pnpm clean:builds      # Remove dist/.next/ only
pnpm prettier:write    # Auto-format all files
pnpm lint:fix          # Auto-fix eslint issues
```

### E2E Tests

End-to-end tests use **Playwright** + **playwright-bdd** (Gherkin `.feature` files). They run against the local environment at `https://testnet.test`.

**Prerequisites**: Full local stack must be running (`pnpm local:setup && pnpm dev`).

```bash
# Install Playwright browsers (once per machine)
pnpm e2e:install

# Run all scenarios headless
pnpm e2e:test

# Run all scenarios with visible browser
pnpm e2e:test:headed

# Run a specific scenario by grep (run from e2e/ directory)
cd e2e && pnpm exec playwright test --grep "auth"
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [interledger/testnet](https://github.com/interledger/testnet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
