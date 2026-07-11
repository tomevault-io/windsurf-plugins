---
trigger: always_on
description: This file provides guidance when working with code in this repository.
---

This file provides guidance when working with code in this repository.

## Workflow

### Before starting work

1. List file names in `docs/` to understand available documentation.
2. Read any docs files relevant to the task at hand.

### Before finishing work

Run all quality gates in order:

```bash
pnpm check:write          # Biome lint + format with auto-fix
pnpm typecheck            # Type-check all packages
pnpm test                 # Unit tests (requires build)
pnpm test:e2e:containers  # E2E tests (requires Docker)
```

### After finishing work

1. List file names in `docs/` again.
2. Create, update, or delete any docs files affected by the changes.

### Code principles

- **DRY** — extract shared logic; never duplicate.
- **Low LOC** — delete dead code, prefer concise expressions, avoid boilerplate.
- **Simplicity** — the smallest change that solves the problem. No speculative abstractions.
- **Long-term maintainability** — clear names, narrow interfaces, minimal coupling.

## Build & Development Commands

```bash
pnpm install              # Install all dependencies
pnpm build                # Build all packages (turbo, respects dependency order)
pnpm typecheck            # Type-check all packages
pnpm test                 # Run unit tests (requires build first; no Docker needed)
pnpm test:e2e:containers  # Run Docker-backed e2e tests (needs container runtime)
pnpm check:write          # Lint + format (Biome) with auto-fix
pnpm lint                 # Lint only (no fix)
pnpm format               # Format check only (no fix)
```

### Running a single test

Tests use Node's built-in test runner via `tsx`. From the toolkit package:

```bash
pnpm --filter @madebywild/agent-harness-framework exec tsx --test test/hooks.test.ts
```

### Watch mode during development

```bash
pnpm --filter @madebywild/agent-harness-framework watch
```

### CLI entrypoint (local dev)

After building: `packages/toolkit/dist/cli.js`

## Architecture

This is a **pnpm monorepo** with two packages, managed by Turborepo:

### `packages/manifest-schema` (`@madebywild/agent-harness-manifest`)

Zod schemas and TypeScript types for the `.harness/` workspace contract: manifest, lock, managed index, overrides, registries, and versioning constants. This is a dependency of the toolkit — changes here require rebuilding both packages.

Key files: `src/index.ts` (all schemas), `src/versioning.ts` (schema version constants).

### `packages/toolkit` (`@madebywild/agent-harness-framework`)

The main package containing the CLI and core engine. Key modules:

- **`src/cli.ts`** — CLI entrypoint; delegates to `src/cli/main.ts` which uses Commander.
- **`src/cli/`** — Command registration, contracts/types, handlers, adapters, and TUI renderers.
- **`src/engine.ts`** — `HarnessEngine` class: the orchestrator for init, plan, apply, watch, doctor, migrate, add/remove, registry operations, and provider enable/disable.
- **`src/loader.ts`** — Loads and validates canonical state: manifest, entities, source files, override sidecars, env var substitution.
- **`src/planner.ts`** — Builds a deterministic plan: drift detection, collision checks, create/update/delete operations, next lock/index computation.
- **`src/repository.ts`** — File I/O: read/write manifest, lock, managed index; atomic file operations.
- **`src/entity-registries.ts`** — Git registry clone, pull, entity materialization from remote registries.
- **`src/registry-validator.ts`** — Validates registry repo structure.
- **`src/hooks.ts`** — Canonical hook parsing and provider-specific projection.
- **`src/env.ts`** — env var placeholder loading and substitution.
- **`src/paths.ts`** — Path resolution for `.harness/` workspace layout.
- **`src/provider-adapters/`** — Per-provider rendering: `claude.ts`, `codex.ts`, `copilot.ts`, plus shared logic for MCP, subagents, hooks, and renderers.
- **`src/versioning/`** — `doctor.ts` (schema health checks), `migrate.ts` (schema migration), `registry.ts` (version registry).
- **`src/engine/`** — Engine sub-modules: `entities.ts` (add/remove/pull), `state.ts` (manifest read), `utils.ts` (config loading/validation).

### Pipeline flow

`loader.ts` → `planner.ts` → `engine.ts` (apply):

1. Validate workspace schema versions (doctor preflight)
2. Load manifest + canonical entities + override sidecars (with env substitution)
3. Render provider artifacts through adapters
4. Detect drift, collisions, creates/updates/deletes
5. `plan` returns diagnostics + operations; `apply` writes files + persists lock/index

## Code Style

- **TypeScript** targeting ES2022 with `NodeNext` module resolution. ESM only (`"type": "module"`).
- **Biome** for linting and formatting (2-space indent, double quotes, semicolons, 120 char line width).
- **Lefthook** pre-commit hooks auto-format and lint staged files; pre-push runs typecheck.
- Strict TypeScript: `noUncheckedIndexedAccess`, `noImplicitOverride`, full `strict` mode.
- **Git** — use [Conventional Commits](https://www.conventionalcommits.org/) format (e.g. `feat:`, `fix:`, `refactor:`, `docs:`, `test:`, `chore:`).

## Release


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebywild/agent-harness](https://github.com/madebywild/agent-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
