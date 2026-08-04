---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Repository Overview

Lunora is a pnpm monorepo for the Lunora framework — a type-safe, real-time backend on Cloudflare Workers + Durable Objects with a Vite-first DX. Packages live under `packages/<name>/`. Apps (examples, docs site, studio) live under `apps/<name>/`.

**Package manager**: pnpm v11.5.3 (enforced via `packageManager`). **Monorepo orchestration**: @visulima/vis. **Node**: ^22.15.0 || >=24.11.0.

## Build & Test Commands

```bash
# Build
pnpm run build                    # All targets (dev)
pnpm run build:packages           # Just packages
pnpm run build:affected           # Only changed projects

# Test
pnpm run test                     # All tests
pnpm run test:coverage            # With coverage
pnpm run test:affected            # Only changed projects

# Single package (use pnpm --filter)
pnpm --filter "@lunora/runtime" run test
pnpm --filter "@lunora/runtime" run lint:types

# Lint
pnpm run lint:eslint              # ESLint all (add :fix to autofix)
pnpm run lint:prettier            # Prettier check (add :fix to autofix)
pnpm run lint:types               # TypeScript type check
pnpm run lint:affected:eslint     # Only changed
pnpm run lint:affected:types      # Only changed
pnpm run lint:package-json        # package.json key order (add :fix to autofix)
```

> **`package.json` key-order gotcha.** Key order is enforced by its own CI job ("Lint (package.json sort)") and by **nothing else** — ESLint, Prettier, `lint:types`, `api:check`, and `dist:check` are all blind to it. So a hand-added block in the wrong position (classically `peerDependencies` placed above `devDependencies` instead of below) goes green locally and red in CI. Canonical order is whatever `vis sort-package-json` emits; run `pnpm run lint:package-json` (= `vis sort-package-json --check`) after editing any manifest.
>
> Note `vis sort-package-json --help` currently crashes ([visulima#741](https://github.com/visulima/visulima/issues/741)) whenever a command's help text contains a literal `{`, so its flags aren't discoverable that way. `--check`, `--sort-scripts`, `--indent`, `--ignore <glob>`, `--sort-order`, `--unsorted <section>`, and `--line-ending` all exist.

> **Stale-`dist` gotcha.** `dist/` is gitignored and built on demand. A raw `pnpm --filter … run test` / `lint:types` does **not** rebuild workspace dependencies, so if an upstream `@lunora/*` package's source changed you may hit stale-`dist` errors (`X is not a function`, "missing export"). Build first — `pnpm run build:packages` once, or `pnpm --filter "@lunora/<pkg>..." run build` (the trailing `...` includes dependencies) — or use `pnpm run test:affected` / `pnpm run lint:affected:types`, which build dependencies for you.

## Commit Convention

Angular-style conventional commits, enforced by hooks:

```
<type>(<scope>): <subject>
```

Types: `feat`, `fix`, `perf`, `docs`, `dx`, `refactor`, `test`, `workflow`, `build`, `ci`, `chore`, `types`, `wip`, `release`, `deps`, `revert`. Scope is typically the package name (e.g., `feat(runtime): add durable-object client`). Subject: imperative, lowercase, no period, max 50 chars. Do not author `release` commits by hand.

## Branch Strategy

- **alpha**: Primary development branch — most PRs target this (default branch)
- **main**: Stable releases
- **next/beta**: Pre-release channels
- Feature branches: `feat/name`, `fix/issue-number`

## Architecture Overview

Lunora exposes a typed, chainable functional API (the `query`/`mutation`/`action` procedure builders) on top of Cloudflare Workers and Durable Objects:

- **Default topology**: a single Durable Object per app — easiest to reason about, sufficient for most apps.
- **Opt-in sharding**: `.shardBy(key)` partitions state across many DOs by user/tenant/room.
- **Opt-in global replication**: `.global()` replicates a function/state across regions for low-latency reads.
- **Vite-first DX**: a Vite plugin powers codegen, server↔client type sync, and the dev server.
- **Type-safe end-to-end**: functions, queries, mutations, and subscriptions infer types from server to client.

## Package Structure

### Naming

The CLI binary is `lunora`. The npm scope is `@lunora/*`. The "main" server package is **`@lunora/server`** (directory `packages/server/`) — it exports `defineSchema`, `query`, `mutation`, `action`, and the function-context types. "Main runtime package" in docs/plans means `@lunora/server`.

There is an unscoped **umbrella** package `lunorash` (directory `packages/lunora/`; npm name is `lunorash` because `lunora` is taken on npm, but the directory and CLI bin stay `lunora`). It re-exports the base packages (`@lunora/server` + subpaths, `@lunora/values`, `@lunora/runtime`, `@lunora/do`, `@lunora/client`) via subpaths (`lunorash/server`, …) and ships the `lunora` CLI bin. Codegen emits `lunorash/*` imports in `_generated/*` when a project declares a `lunorash` dependency (else `@lunora/*`) — opt-in and backward-compatible. Add-ons/adapters/Vite plugin stay separate installs.

### Packages


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anolilab/lunora](https://github.com/anolilab/lunora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
