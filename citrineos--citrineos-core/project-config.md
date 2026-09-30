---
trigger: always_on
description: Instructions for automated coding agents working in `citrineos-core`. Everything here applies to the whole
---

# AGENTS.md

Instructions for automated coding agents working in `citrineos-core`. Everything here applies to the whole
repository unless a more specific `AGENTS.md` exists in a subdirectory. Human-facing documentation lives in
[README.md](./README.md).

If you are an agent and you have read this file, follow it. If an instruction here conflicts with something you
inferred from the code, the instruction here wins; if it conflicts with a direct request from the user, the user
wins.

## What this repository is

A **pnpm monorepo** (TypeScript, Node) containing the CitrineOS charging station management system: OCPP message
routing and handling, the persistence layer, the OCPI server, and the operator web UI. Charging stations connect
over WebSocket; modules talk to each other through RabbitMQ; data lives in PostgreSQL; the UI reads and writes through Hasura while messages triggered by the UI are routed to specific endpoints that handle them. 

## Environment and setup

- Node and pnpm versions are pinned in the repository: `.nvmrc` for Node, `packageManager` in the root
  `package.json` for pnpm.
- Install once at the root: `pnpm install`. Never `npm install` or `yarn` — the lockfile is pnpm's and the
  workspace depends on pnpm's linking and catalog.
- Docker is optional for unit work, required for the full stack and for the testcontainers-backed suites.

## Commands

Run these from the repository root.

| Task                           | Command                                          |
| ------------------------------ | ------------------------------------------------ |
| Build everything               | `pnpm build`                                     |
| Build one package and its deps | `pnpm --filter "@citrineos/<name>..." run build` |
| Run tests                      | `pnpm test`                                      |
| Tests with coverage            | `pnpm test:coverage`                             |
| Typecheck test sources         | `pnpm typecheck:test`                            |
| Lint                           | `pnpm lint` / `pnpm lint:fix`                    |
| Format                         | `pnpm prettier`                                  |
| Remove build artifacts         | `pnpm clean`                                     |
| Bring the Docker stack up      | `pnpm citrine` (see README for flags)            |

**Do not run `tsc -b tsconfig.build.json`.** It is a base config with no `outDir`; invoking it directly emits
thousands of `.js`/`.d.ts` files next to the sources and pollutes the working tree. Use `pnpm build`, which runs
each package's own build script. If a tree ever gets polluted this way, the stray files sit untracked next to
the sources — preview them with `git clean -nd`, then delete them.

## Verifying your work

Before reporting a change as complete:

1. `pnpm build` — the workspace compiles.
2. `pnpm test` — the suite passes.
3. `pnpm lint`, then `pnpm exec prettier --check` on the files you changed — CI runs the linter on every pull
   request, and `eslint-plugin-prettier` makes formatting drift a lint error. `pnpm prettier` formats the entire
   repository, so reach for it only to fix your own files.

Some suites use [testcontainers](https://testcontainers.com/) and need a running Docker daemon. Without one they
fail at container startup, before any assertion runs. Read which of the two you are looking at: a container that
never started tells you nothing about your change; a container that started and then failed an assertion tells
you something.

## Repository layout

```
citrineos-core/
├── apps/
│   ├── ocpp-server/     # OCPP server entrypoint, Docker setup, migrations, EVerest harness
│   ├── ocpi-server/     # OCPI server
│   ├── operator-ui/     # Operator web UI — Next.js + Refine, Playwright e2e
│   └── mock-msp/        # Mock MSP used for OCPI scenarios
├── packages/
│   ├── types/           # Shared types: OCPP model types, DTOs, Zod schemas, enums (no runtime)
│   ├── base/            # Shared interfaces, config, utilities
│   ├── dal/             # Persistence — models, repositories, mappers
│   ├── ocpp/            # OCPP modules, handlers, endpoints, transport
│   └── ocpi/            # OCPI modules, handlers, endpoints, mappers, transport
├── scripts/stack.mjs    # Docker stack launcher
└── pnpm-workspace.yaml  # Workspace members + the dependency catalog
```

Each workspace member has its own README with component-specific detail; read the one for the area you are
changing before you change it.

## Conventions you must follow

**License headers.** Code files carry an SPDX header — TypeScript, JavaScript and YAML.

```ts
// SPDX-FileCopyrightText: 2026 Contributors to the CitrineOS Project
//
// SPDX-License-Identifier: Apache-2.0
```

**Generated OCPP schemas for PRs are off-limits.** The model schemas and types under `packages/types/src/ocpp/model` are

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [citrineos/citrineos-core](https://github.com/citrineos/citrineos-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
