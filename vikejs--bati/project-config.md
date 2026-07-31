---
trigger: always_on
description: Bati is a next-generation scaffolding CLI tool for the Vike (Vite-based) ecosystem. It generates fully-functional starter apps by combining boilerplates for different features (React/Vue/Solid, servers, databases, auth, etc.).
---

# Copilot Instructions for Bati Repository

## Overview

Bati is a next-generation scaffolding CLI tool for the Vike (Vite-based) ecosystem. It generates fully-functional starter apps by combining boilerplates for different features (React/Vue/Solid, servers, databases, auth, etc.).

**Repository Structure:**
- **TypeScript monorepo** managed with **bun workspaces** and **Nx**
- Node.js ≥22 required, Bun ≥1.3.11 (`packageManager` pins `bun@1.3.11`)
- Workspaces across `/packages/` (10 packages) and `/boilerplates/` (~50 folders, ~42 user-facing features)

## Build Commands (Execute in Order)

**ALWAYS run these commands from the monorepo root:**

```bash
# 1. Install dependencies (required before any build)
bun install

# 2. Build all packages (~55 seconds)
bun run build

# 3. Run unit tests (~15 seconds)
bun run test

# 4. Run type checking (~60 seconds)
bun run check-types

# 5. Run linting (Biome)
bun run lint
```

**Important Notes:**
- `bun run build` must be run after `bun install` and before tests or CLI
- Build uses Nx caching; use `bun run build:force` to rebuild without cache
- The `format` step runs automatically after build via Biome

## Testing

```bash
# Unit tests (fast, ~15s)
bun run test

# E2E tests — every combo (extensive; CI runs these, slow locally)
bun run test:e2e

# Run only the combos whose flags match (superset filter)
bun run test:e2e --react --trpc

# Run one exact combo — generated and run even if matrix.ts doesn't list it
bun packages/tests/e2e/runner.ts exact --react --hono --trpc --sqlite --drizzle --eslint --biome --oxlint

# Restrict any run to specific checks (skips the server boot) — and rerun what last failed
bun packages/tests/e2e/runner.ts exact --react --hono … --check=typecheck,knip
bun packages/tests/e2e/runner.ts failed

# The generated apps are removed when a run ends; pass --keep to inspect them afterwards
bun packages/tests/e2e/runner.ts exact --react --hono … --keep

# Print the selection without running it; or emit the matrix JSON the CI fan-out consumes
bun packages/tests/e2e/runner.ts all --react --dry-run
bun packages/tests/e2e/runner.ts list
```

## Adding E2E Tests for New Features

When adding a new feature, **add E2E tests** to verify it works correctly.

### E2E Test Structure

All E2E tests live in `packages/tests/e2e/` — one code path for local and CI:

| File | Responsibility |
|---|---|
| `matrix.ts` | the single declaration of which combos exist — an array of `suite()` builders |
| `runner.ts` | matrix → generate an app per combo → run them all as Vitest projects (`createVitest`) |
| `fixtures.ts` | boot/teardown the app in a mode, the `fetch` test fixture, shared helpers |
| `e2e.spec.ts` | every assertion, shared by all projects (each self-gates on flags via `BATI.has(...)`) |

A combo runs in up to three passes (`e2e.spec.ts`): a **primary** pass (boot in its `mode`, run every assertion), an optional **smoke** pass (re-run `/` once built/containerized — `.kind(...)` combos), and **checks** (lint / typecheck / knip).

### Declaring combos

`matrix.ts` exports an array of `suite()` builders (from `@batijs/tests-utils`, defined in `packages/tests-utils/src/suite.ts`). This **include-only** builder replaced the old `matrix` + `exclude` exports:

```ts
import { framework, server, spread, suite } from "@batijs/tests-utils";

const matrix = [
  // Cross product of named axes. `null` = "this axis is absent in that combo".
  // `.matrix(...)` can be called multiple times — each call unions in more combos.
  suite()
    .matrix({
      framework: ["react", "vue"],          // one combo per listed value
      server: "hono",                        // single value = always present
      data: ["trpc", "telefunc", "ts-rest", null],
    })
    .linters("eslint", "biome")              // flags appended to every combo
    .kind("data"),                           // tags the suite (see below)
];

export default matrix;
```

Key `suite()` API (see `suite.ts` for the full surface):
- **Axes** are derived from feature categories and stay in sync with `features.ts`: `framework`, `server`, `data`, `db`, `orm`, `deploy`, `css`, `auth`, `analytics`.
- `.matrix({...})` — cross product of named axes; call it repeatedly to add unions of combos (this is how you express "exclusions" — enumerate exactly the combos you want instead of subtracting). `null` replaces the old `undefined` "without" sentinel.
- `.case({...})` — one explicit combo (values can be arrays, e.g. `flags: ["sentry", "logrocket"]`).
- `.linters(...)` — flags added to every combo.
- `.mode(m)` — the combo's primary run mode: `"dev"` (default), `"prod"`, `"preview"`, `"docker"`, or `"none"` (file checks only, no server).
- `.kind(k)` — tags a suite as `"data"`, `"auth"`, or `"cloudflare"`; this enables the kind-scoped assertions in `e2e.spec.ts` and a built/containerized smoke pass (`docker` for dokploy, `preview` for cloudflare, else `prod`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vikejs/bati](https://github.com/vikejs/bati) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
