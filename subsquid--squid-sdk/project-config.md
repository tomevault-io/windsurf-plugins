---
trigger: always_on
description: > **Scope:** Root repository ([subsquid/squid-sdk](https://github.com/subsquid/squid-sdk)). Applies to all subdirectories unless a future nested `AGENTS.md` explicitly overrides (none exist today).
---

# Project Agent Guide

> **Scope:** Root repository ([subsquid/squid-sdk](https://github.com/subsquid/squid-sdk)). Applies to all subdirectories unless a future nested `AGENTS.md` explicitly overrides (none exist today).
>
> **Quality tooling:** This repo uses **[Biome](https://biomejs.dev/)** for **linting and formatting**, and **[Vitest](https://vitest.dev/)** for **unit tests** (via package `test` scripts and the shared root `vitest.config.ts`).

## Quick Facts

| Item | Detail |
| --- | --- |
| **Primary language** | TypeScript |
| **Monorepo tool** | [Rush](https://rushjs.io/) **5.170.1** (invoke via `node common/scripts/install-run-rush.js` or global `rush`) |
| **Package manager** | **pnpm** **9.15.4** (Rush-managed); **`strictPeerDependencies`** and shrinkwrap validation are enforced — fix peers in `package.json`, then **`rush update`** |
| **Node** | `rush.json` allows **>=18.13.1**; CI (`.github/workflows/test.yml`) uses **Node 24** |
| **Project paths** | Packages are registered in `rush.json`; folders are **two levels** from repo root (`category/project-name`, e.g. `evm/evm-stream`) |
| **Linting & formatting** | **[Biome](https://biomejs.dev/)** — config in `biome.json`; run through **`rush lint`**, **`rush format`**, **`rush biome`**; tool pinned in `common/autoinstallers/lint/` |
| **Unit testing** | **[Vitest](https://vitest.dev/)** — shared `vitest.config.ts`; packages run it from their **`test`** script; runner pinned in `common/autoinstallers/vitest/` |
| **Shrinkwrap** | `common/config/rush/pnpm-lock.yaml` — edit deps with **`rush add` / `rush remove`**, then **`rush update`**; use **`rush install`** read-only on CI |
| **Version policy** | **`ensureConsistentVersions`** is **on** — align semver ranges across packages or allow exceptions in `common/config/rush/common-versions.json`; run **`rush check`** |

Official Rush references: [Rush overview](https://rushjs.io/), [new developer guide](https://rushjs.io/pages/developer/new_developer/), [RushStack hub](https://rushstack.io/).

## Repository Tour

Top-level layout (domains are grouped as `category/package`; publishable packages are mostly `@subsquid/*`):

| Path | Role |
| --- | --- |
| `bitcoin/`, `evm/`, `fuel/`, `solana/`, `starknet/`, `substrate/`, `tron/` | Chain-specific libraries and tooling |
| `graphql/` | GraphQL server and OpenReader |
| `hyperliquid/` | Hyperliquid-related packages |
| `typeorm/` | TypeORM integration packages |
| `util/` | Shared utilities and internal helpers |
| `processor/` | Batch processor |
| `ops/` | Internal automation (e.g. `workspace` — dependency unify script used by `rush unify-dependencies`) |
| `test/` | Example squid projects, benches, and test harness packages (not all published) |
| `common/` | Rush config (`common/config/rush/`), autoinstallers, scripts, change files (`common/changes/`), temp install/build state (`common/temp/`) |
| `test/e2e-suite/` | End-to-end shell driver for **`rush e2e`** |

There is **no** repo-wide `Justfile`. **Makefiles** exist under **individual** packages (mostly `test/*` and a few libraries) for local convenience — not the primary workflow entrypoint.

**Layout snapshot (directories only, depth 1):** `.github/`, `.agents/`, `bitcoin/`, `common/`, `evm/`, `fuel/`, `graphql/`, `hyperliquid/`, `ops/`, `processor/`, `solana/`, `starknet/`, `substrate/`, `test/`, `tron/`, `typeorm/`, `util/` — plus root config (`rush.json`, `biome.json`, `vitest.config.ts`, `Dockerfile`, etc.).

## Tooling & Setup

1. **Clone:** Large fixtures may use Git LFS. To skip LFS smudge:  
   `GIT_LFS_SKIP_SMUDGE=1 git clone git@github.com:subsquid/squid-sdk.git`
2. **Rush:** Prefer pinned invocations:  
   `node common/scripts/install-run-rush.js <command>`  
   (or `npm install -g @microsoft/rush` per [CONTRIBUTING.md](CONTRIBUTING.md).)
3. **Install then build:**  
   `rush install` then `rush build` (see **Rush command discipline** below for scoped commands).
4. **Docker:** Required for some tests (see [CONTRIBUTING.md](CONTRIBUTING.md)).
5. **Editor tooling:** After `rush install`, Biome and Vitest are linked at repo root `node_modules/` for extensions — avoid **phantom dependencies** via a stray root `node_modules`; Rush warns if it finds one.

**Secrets / env:** No single `.env.example` at root is required for the core Rush workflow; individual packages or test apps may document their own. Prefer package READMEs and [Subsquid docs](https://docs.subsquid.io/) for runtime setups.

### Rush Command Discipline

Follow [Rush best practices](https://rushjs.io/): treat this repo as Rush-first.

| Practice | Detail |
| --- | --- |
| **Prefer Rush entrypoints** | Use **`rush`** for repo-wide operations, **`rushx`** for one package’s scripts (same as `npm run`, Rush-aware). Avoid **`npm` / `pnpm` / `yarn`** for installing or linking workspace packages; that bypasses Rush and breaks guarantees. |
| **When you need pnpm** | Use **`rush-pnpm`** so PNPM runs with the correct workspace context. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [subsquid/squid-sdk](https://github.com/subsquid/squid-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
