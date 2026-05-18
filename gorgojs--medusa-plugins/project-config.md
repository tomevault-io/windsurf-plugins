---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Repository Structure

This repository is organized as a Yarn v4 monorepo with shared workspaces for plugin packages.

```text
├── examples/
│   ├── 1c/
│   ├── feed-yandex/
│   ├── fulfillment-apiship/
│   ├── payment-robokassa/
│   └── payment-tkassa/
├── packages/
│   ├── medusa-1c/
│   ├── medusa-feed-yandex/
│   ├── medusa-fulfillment-apiship/
│   ├── medusa-payment-robokassa/
│   └── medusa-payment-tkassa/
├── scripts/
└── www/
    └── docs/
```

## Essential Commands

### Monorepo Commands

```bash
yarn install                          # Install all dependencies (triggers husky setup)

 # Update Medusa version across examples
yarn update <version> [example] [-s|--single] [--skip-build]
# e.g.: yarn update 2.14.0 feed-yandex --single --skip-build

# Changesets (versioning & publishing)
yarn changeset                        # Interactively create a changeset
yarn changeset version                # Bump versions and update CHANGELOGs
yarn changeset publish                # Publish packages to npm
```

### Per-Package (Per-Plugin) Commands

Run inside `packages/<name>/`:

```bash
# development
yarn dev                              # Publish locally then start dev watch mode
```

### Per-Example Commands

Run inside `examples/<name>/medusa`:

```bash
yarn install                          # Install all dependencies 
yarn dev                              # Run in development mode
yarn dev:tunnel                       # Run in development mode with tunneling

# testing
yarn test:integration:http            # HTTP integration tests 
yarn test:integration:modules         # Module integration tests
yarn test:unit                        # Unit tests
```

Run inside `examples/<name>/medusa-storefront`:

```bash
yarn install                          # Install all dependencies 
yarn dev                              # Run in development mode
yarn dev:tunnel                       # Run in development mode with tunneling
```

## Commit Conventions

Commits must follow [Conventional Commits](https://www.conventionalcommits.org/) — enforced by commitlint.

**Scope is required** and must be one of:

- Package scope (strip `medusa-` prefix):
  - `1c`
  - `feed-yandex`
  - `fulfillment-apiship`
  - `payment-robokassa`
  - `payment-tkassa`
- Repo-level:
  - `deps`
  - `release`
  - `docs`
  - `root`

Examples:

```
feat(feed-yandex): add price filter support
fix(payment-robokassa): handle webhook timeout
chore(deps): bump @medusajs/medusa to 2.14.0
```

Scope maps directly to changeset bump type: `feat` → minor, `fix/perf/refactor/docs/revert/test` → patch, breaking (`!`) → major.

## Package Architecture

### Source structure (inside package/<name>/src/)

Not every package uses every directory — include only what the plugin needs.

```
src/
├── admin/          # React admin UI
│   ├── routes/         # UI-routes extending Medusa Admin
│   ├── components/     # shared UI
│       └── /gorgo-widgets      # Components for Gorgo UI-widgets
│   ├── hooks/          # React hooks
│   ├── lib/            # admin-only helpers (e.g. sdk.ts)
│   ├── widgets/        # UI-widgets extending Medusa Admin
│   └── i18n/           # i18next JSON locales
├── api/            # REST route handlers
│   ├── admin/          # authenticated admin endpoints
│   └── {store,hooks}/  # public / webhook endpoints
├── modules/        # Medusa modules
│   └── <name>/
│       ├── index.ts            # Module(<NAME>, { service })
│       ├── service.ts          # extends MedusaService({ Model })
│       ├── models/             # model.define(...)
│       ├── migrations/         # generated migrations
│       └── loaders/            # optional module loaders
├── providers/      # Medusa providers (payment / fulfillment / ...)
│   └── <name>/
│       ├── index.ts            # ModuleProvider(Modules.X, { services })
│       └── service.ts          # extends AbstractXProvider
├── workflows/      # Medusa workflows + steps
│   ├── <name>/
│   │   ├── index.ts            # createWorkflow(...)
│   │   └── steps/              # createStep(...)
│   └── index.ts                # barrel re-export
├── gorgo-widgets/  # Gorgo UI-widgets extending Gorgo plugins
├── jobs/           # scheduled background jobs
├── lib/            # cross-cutting libraries used by the package
├── utils/          # small utility helpers
├── data/           # static data / fixtures / seed files
└── types/          # shared TypeScript types (barrel via index.ts)
```

### TypeScript

All packages target ES2021, Module Node16, with SWC for ts-node. Build output includes both `.js` (CJS) and `.mjs` (ESM) for the admin bundle.

## MCP

Use the **medusa** MCP (`docs.medusajs.com`) for Medusa v2 APIs and patterns, and **context7** (shipped in `.mcp.json`) for third-party SDK docs — prefer both over memory.

## CI/CD

| Workflow | Trigger | Purpose |
|---|---|---|
| `publish.yml` | Push to main (packages/**) | Auto-generate changesets → version → publish to npm |
| `update-medusa-version.yml` | Daily 6 AM + manual | Check latest Medusa, run integration tests, open update PR |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gorgojs/medusa-plugins](https://github.com/gorgojs/medusa-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
