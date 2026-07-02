---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`Weft` is a pnpm monorepo (`weft-workspace`) implementing an Effect-based UI library with strict TypeScript configuration and modern tooling.

Workspace layout (see `pnpm-workspace.yaml`):

- `packages/*` — published library packages:
  - `@weftui/base` (`packages/base`) — shared primitives
  - `@weftui/core` (`packages/core`) — core combinators, sources, streams, boundaries
  - `@weftui/dom` (`packages/dom`) — DOM renderer with `./client` and `./server` entry points
- `examples/*` — standalone runnable example apps, each its own workspace package

## Requirements

- Node.js

See versions in package.json > engines. Package management and all tooling is handled by `vp` (Vite+).

## Development Commands

All commands use the `vp` CLI (Vite+). Run `vp help` for a full list.

### The `pack` step (read this first)

This is a monorepo: `@weftui/dom` and the `examples/*` consume `@weftui/core`/`@weftui/base` as workspace packages, resolved through their **built `dist/`**. Cross-package type-checking is therefore only correct once those packages have been packed.

**Rule: run validation through the `vp run <task>` tasks, never the bare `vp <command>`.** The tasks are declared in the root `vite.config.ts` under `run.tasks` and each one declares `dependsOn: ["pack"]`, so `vp run` always rebuilds the packages first:

- ✅ `vp run check`, `vp run test`, `vp run test:browser` — pack first, then run. Always correct.
- ❌ `vp check`, `vp test` directly — skip `pack`, so against stale/missing `dist/` they report **false** cross-package errors (e.g. spurious `implicit any` from unresolved `@weftui/*` types). Only safe right after a pack.

Treat the task list in `vite.config.ts` (`run.tasks`) as the source of truth for how to validate — if a task exists there, invoke it via `vp run <task>`. Current tasks: `dev`, `pack`, `check`, `test`, `test:browser`.

### Building

```bash
vp build
```

Uses tsdown for fast TypeScript bundling.

### Testing

```bash
vp run test            # Pack, then run all node/jsdom tests
vp run test:browser    # Pack, then run real-browser e2e tests (Playwright)
vp test --watch        # Watch mode (only safe after a pack)
```

Uses Vitest (via Vite+). Node test files follow the pattern `**/*.test.{ts,tsx}`; `*.browser.test.{ts,tsx}` are excluded from `vp run test` and run via `vp run test:browser` (see the `pack` step rule above).

### Checking (format + lint + typecheck)

```bash
vp run check       # Pack, then format, lint, and type-check all files
vp check --fix     # Auto-fix formatting/lint (only safe after a pack)
```

**Important:** Validate via `vp run check` (it packs first — see the `pack` step rule). Use `vp check --fix` for auto-fixing, but only when packages are already built, otherwise it reports false cross-package type errors. Always prefer these over individual lint/format commands.

## Architecture

### TypeScript Configuration

Strict TypeScript setup with:

- `noUncheckedIndexedAccess: true` - Array/object access returns possibly undefined
- `noImplicitReturns: true` - All code paths must return
- `strict: true` - All strict type-checking enabled
- `verbatimModuleSyntax: true` - Import/export syntax preserved
- `isolatedModules: true` - Each file must be transpilable independently
- `noUncheckedSideEffectImports: true` - Side-effect imports must be explicit

Path aliases (configured per package in `packages/*/tsconfig.json`, which extend `tsconfig.base.json`):

- `~/*` maps to that package's `./src/*`

### Code Style

**Toolchain:** This project uses Oxlint (linting) and Oxfmt (formatting) via Vite+, NOT ESLint or Biome.

Oxfmt enforces:

- Tab indentation
- Double quotes for strings

When ignoring lint rules, use Oxlint syntax:

- ✅ Correct: `// oxlint-disable-next-line <rule-name>`
- ❌ Wrong: `// eslint-disable-next-line` or `// biome-ignore`

### Project Structure

- `packages/*/src/` - Source TypeScript files for each library package
- `packages/*/dist/` - Build output (excluded from TypeScript compilation)
- `examples/*/` - Standalone runnable example apps, each its own workspace package with an `app.ts` entry point and `vite.config.ts`
- `docs/` - Documentation
- `plans/` - Design plans and specs
- ES modules only (`"type": "module"` in package.json)

### Examples

The `examples/` folder contains standalone workspace packages demonstrating specific patterns or features (e.g. `keyed-list`, `form-handling`, `ssr-hydration`).

**Rules for examples:**

- Every example must have a co-located README named `readme.md`
- Each example is a self-contained, runnable workspace package (depends on `@weftui/*` via `workspace:*`)
- Include a JSDoc header comment in `app.ts` explaining the example's purpose
- READMEs should include: Overview, Problem, Solution, How It Works, and When to Use sections
- Each example is split into a **side-effect-free `app.ts`** (or `src/app.ts`) that
  `export`s `App` — no top-level `mount`/`hydrate` call — and a thin entry
  (`main.ts`, or `entry-client.ts` for SSR examples) that mounts it and is the file
  referenced by `index.html`. This keeps `app.ts` importable by tests.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stefvw93/weft](https://github.com/stefvw93/weft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
