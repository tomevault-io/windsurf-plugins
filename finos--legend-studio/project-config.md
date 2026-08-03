---
trigger: always_on
description: This document is designed to help AI agents understand the structure, workflows, and conventions of the `legend-studio` repository.
---

# Legend Studio - Agent Guide

This document is designed to help AI agents understand the structure, workflows, and conventions of the `legend-studio` repository.

## Project Overview

`legend-studio` is the codebase for Legend applications, including Legend Studio, Legend Query, Legend DataCube, Legend Marketplace, Legend Pure IDE, and Legend REPL. It is a monorepo managed with Yarn workspaces.

## Environment Setup

### Prerequisites

- **Node.js** `>= 20`
- **Yarn 4.x** (Berry). The repo pins `packageManager` to `yarn@4.x` — enable via Corepack: `corepack enable`. Do **NOT** use `npm` or `pnpm`; doing so will corrupt the `yarn.lock`.
- **Docker** (only required for tests/dev that talk to the Engine or other backend servers).

### Install

```bash
yarn install
yarn setup   # runs per-workspace setup + full build
```

## Development Workflow

### Running Applications

Each application has a dedicated `dev:*` script that runs the corresponding `*-deployment` workspace:

- **Legend Studio**: `yarn dev` (alias for `yarn dev:studio`)
- **Legend Query**: `yarn dev:query`
- **Legend DataCube**: `yarn dev:datacube`
- **Legend Marketplace**: `yarn dev:marketplace`
- **Legend Pure IDE**: `yarn dev:pure`
- **Legend REPL**: `yarn dev:repl`

Useful companions while developing:

- `yarn dev:ts` — TypeScript project-references watch (rebuilds `.d.ts` across packages).
- `yarn dev:sass` — Sass watch.
- `yarn dev:mock-server` / `yarn dev:mock-depot-server` — lightweight mock backends (avoids needing real servers).
- `yarn dev:assemblage` — wire a local dev assemblage. See [docs/workflow/local-development-assemblage.md](docs/workflow/local-development-assemblage.md).

### Scoping commands to a single workspace

This is a Yarn workspaces monorepo. Run any package script against a single workspace with:

```bash
yarn workspace @finos/<package-name> <script>
# e.g.
yarn workspace @finos/legend-query-builder test
yarn workspace @finos/legend-graph build
```

Most top-level scripts (`lint`, `test`, `build`) fan out across all workspaces in parallel.

### Building

- `yarn build` — full clean + TypeScript + Sass build.
- `yarn build:ts` — TypeScript only.
- `yarn clean` — remove `build/` and per-package artifacts.
- `yarn clean:cache` — also clears the Jest cache. Run this after touching project references or `package.json` dependencies.

### Testing

- `yarn test` — run all tests.
- `yarn test:group <name>` — run a specific group (groups are defined per package; list them with `yarn test:list-groups`).
- `yarn test:watch` — watch mode.
- `yarn workspace @finos/<pkg> test` — tests for one package.

Test strategy reference: [docs/technical/test-strategy.md](docs/technical/test-strategy.md).

#### Running Engine Server

Some integration tests (roundtrip / grammar) require a running Engine Server. Start it with Docker:

1. `cd fixtures/legend-docker-setup/grammar-test-setup`
2. `docker compose --file=grammar-test-setup-docker-compose.yml up --detach`

For unit-level work, prefer the in-repo mock server in [fixtures/legend-mock-server](fixtures/legend-mock-server) (`yarn dev:mock-server`).

### Linting, Formatting, and CI Checks

- `yarn lint` — ESLint + Stylelint across all workspaces.
- `yarn check:ci` — full CI gate: `check:pkg` + `check:ts` + `check:format` + `check:copyright` + `check:changeset`.
- Individual checks: `yarn check:ts`, `yarn check:format`, `yarn check:copyright`, `yarn check:pkg`.
- `yarn fix` — auto-fix copyright headers, styles, JS, and formatting.
- `yarn fix:copyright` — insert the Apache 2.0 header into any new source file (see below).

## Project Structure

- [packages/](packages) — all workspace packages (see taxonomy below).
- [scripts/](scripts) — automation: `workflow/`, `release/`, `copyright/`, `test/`, `docker/`, `website/`, `github-bot/`.
- [docs/](docs) — `design/`, `technical/`, `workflow/`, `ux/` documentation.
- [fixtures/](fixtures) — Docker setups and mock servers used by tests and local dev.
- [setup.js](setup.js) — bootstrap helper used by `yarn setup`.

### Package taxonomy

The flat [packages/](packages) directory is large. Quick legend for picking the right package:

- `legend-application-<app>` — application shell (studio, query, datacube, marketplace, pure-ide, repl).
- `legend-application-<app>-bootstrap` — shared bootstrap wiring for that app.
- `legend-application-<app>-deployment` — webpack/vite entry points; this is what `yarn dev:*` runs.
- `legend-extension-dsl-*` / `legend-extension-store-*` — pluggable DSL / store extensions. See [docs/technical/extension-authoring.md](docs/technical/extension-authoring.md) and [docs/technical/instructions-add-plugin.md](docs/technical/instructions-add-plugin.md).
- `legend-server-*` — typed clients for backend services (SDLC, Depot, Lakehouse, Marketplace, Showcase).
- `legend-graph` — PURE metamodel + protocol layer (core).
- `legend-query-builder`, `legend-data-cube`, `legend-code-editor`, `legend-lego`, `legend-art` — shared UI / feature libraries.
- `legend-shared`, `legend-storage`, `legend-dev-utils` — cross-cutting utilities.
- `babel-preset`, `eslint-plugin`, `stylelint-config` — internal tooling configs.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [finos/legend-studio](https://github.com/finos/legend-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
