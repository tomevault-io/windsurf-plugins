---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Alepha is a convention-driven TypeScript framework for building robust, end-to-end type-safe applications.
This is a monorepo workspace using Yarn workspaces with the following structure:

- `apps/*` - Example applications
- `packages/*` - Framework workspace

## Environment Variables for Commands

When running Alepha CLI commands (build, dev, etc.), use these environment variables for verbose output:
- `LOG_FORMAT=pretty` - Human-readable colored log output
- `LOG_LEVEL=trace` - Maximum verbosity (trace, debug, info, warn, error)

Example:
```bash
LOG_FORMAT=pretty LOG_LEVEL=trace yarn w @alepha/devtools build
```

## Development Commands

### Core Commands
- `yarn v` or `yarn alepha verify` - Full verification pipeline: clean, lint, typecheck, test, check-dependencies, build, e2e, clean. **Must complete within 5 minutes** — always run it with a 5-minute timeout. If it exceeds 5 minutes, treat that as a failure (a hung step, usually e2e) and investigate, do not just wait longer.
- `yarn clean` or `yarn alepha clean` - Remove all generated files and node_modules
- `yarn build` - Build all workspace packages using `tsdown`
- `yarn test` - Run all tests using Vitest
- `yarn lint` - Format and lint using Biome (with `--fix` flag)
- `yarn typecheck` - TypeScript type checking (`tsc --noEmit`)
- `yarn check-dependencies` - Check for unused dependencies using depcheck

### Workspace Commands
- `yarn w <workspace> <command>` - Run commands in specific workspace
  - Examples:
    - `yarn w alepha test` - Run tests for alepha package
    - `yarn w @alepha/ui typecheck` - Type check @alepha/ui package
    - `yarn w @alepha/devtools build` - Build @alepha/devtools package

## Architecture

### Framework Core
- Uses primitive-based architecture with `$` prefixed primitives (`$action`, `$entity`, `$repository`, etc.)
- Dependency injection container in `alepha`
- Convention-driven with minimal configuration
- Documentation: https://alepha.dev/llms.txt

### Package Organization

Alepha uses a hybrid monorepo structure:

**Unified Package (`alepha`)**
- The `alepha` package exports 50+ framework sub-modules
- Sub-modules can be imported as `alepha/module-name/submodule-name` (e.g., `alepha/server`, `alepha/security`, `alepha/api/users`)
- Provides unified dependency management and consistent versioning
- Located in `packages/alepha/src/` with each sub-module as a directory

**Specialized Packages**
- `@alepha/ui` - Shared shadcn-based UI for monorepo apps (sourced from `@alepha/ui-registry`)
- `@alepha/ui-registry` - shadcn registry source (blocks distributed via `https://alepha.dev/r/*`)

**@alepha/ui ↔ @alepha/ui-registry workflow**
- `@alepha/ui-registry` is the **source of truth** for all custom Alepha components (controls, admin blocks, auth pages, etc.)
- `@alepha/ui` is the **monorepo consumer** — it pulls components from the registry via `shadcn add`
- **Never manually create/edit component files in `@alepha/ui`** that originate from the registry. Use the shadcn CLI instead.
- To sync registry components into `@alepha/ui`:
  1. Build the registry: `yarn w @alepha/ui-registry build`
  2. Serve the built output: `cd apps/docs/public && python3 -m http.server 8765` (or run the docs dev server)
  3. From `packages/@alepha/ui`, run: `npx shadcn add @alepha/<component-name>`
  4. **Known issue**: `alepha` is a peer dep in `@alepha/ui`, so before running `shadcn add`, temporarily remove `alepha` from `peerDependencies` in `packages/@alepha/ui/package.json`, then restore it after.
  5. **Known issue**: shadcn may rewrite `@/components/ui/X` imports as `@alepha/components/ui/X` (missing `/ui` segment). After adding, verify imports use `@alepha/ui/components/ui/X` and fix with: `sed -i '' 's|@alepha/components/|@alepha/ui/components/|g' <files>`
- To add a **new** component: create it in `@alepha/ui-registry/registry/default/<name>/`, register it in `registry.json`, then `shadcn add` it into `@alepha/ui`
- The `components.json` in `@alepha/ui` configures the `@alepha` registry at `http://localhost:8765/r/{name}.json`
- `@alepha/devtools` - Development tools and inspection UI
- `@alepha/payments-stripe` - Stripe payments backend
- `@alepha/protobuf` - Protocol Buffers support

### Testing

#### Test Configuration
- Uses **Vitest** with global test environment
- Coverage tracking for `packages/*/src/**/*.ts(x)`
- Test databases and Azure storage emulator configuration included via `vitest.config.ts`
- Tests located in `__tests__/` directories within each package / module or as co-located `*.spec.ts` files

#### Test Environments
Two test environments are configured:
1. **Node.js tests** - `*.spec.{ts,tsx}` (excludes `*.browser.spec.*`)
2. **Browser tests (jsdom)** - `*.browser.spec.{ts,tsx}`
   - Use `.browser.spec.ts` or `.browser.spec.tsx` extension for browser tests
   - Automatically uses jsdom environment

#### Running Tests
- **All packages**: `yarn test`
- **Single package**: `yarn w alepha test`
- **Filtered tests**: `yarn w alepha vitest run <pattern>` (e.g., `yarn w alepha vitest run init.spec`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [feunard/alepha](https://github.com/feunard/alepha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
