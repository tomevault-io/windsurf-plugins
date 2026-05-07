---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**svstate** is a Svelte 5 library that provides a supercharged `$state()` with deep reactive proxies, validation, snapshot/undo, and side effects. It's designed as a peer dependency for Svelte 5 projects.

**Module Format:** ESM (ES Modules) only — no CommonJS build is provided.

## Development Commands

**Requires Node >=20, npm >=9**

### Testing

```bash
npm test                              # Run all tests once
npm run test:coverage                 # Run tests with coverage report
npx vitest run test/validators.test.ts  # Run a single test file
npx vitest run -t "should trim"       # Run tests matching pattern
npx vitest                            # Watch mode (re-runs on file changes)
```

Coverage thresholds are set at 60% for lines, functions, branches, and statements.

### Building

```bash
npm run build           # Clean build (tsc --build --clean && --force)
npm run clean           # Clean TypeScript build artifacts
```

### Code Quality

```bash
npm run fix             # Run format → lint → format (recommended)
npm run all             # Run fix → build → test → demo:build (full validation)
npm run lint:check      # Check for linting errors
npm run lint:fix        # Auto-fix linting errors
npm run format:check    # Check code formatting
npm run format:fix      # Auto-fix formatting issues
```

### Demo

```bash
npm run demo            # Start Vite dev server with demo app (in demo/ directory)
```

## Demo Subproject

The `demo/` directory is a separate npm project for interactive testing of the library.

**Structure:**

- `demo/src/App.svelte` - Root component
- `demo/src/pages/` - Demo pages (e.g., `BasicValidation.svelte`)
- `demo/src/components/` - Shared UI components (e.g., `ErrorText.svelte`)

**Stack:** Vite + Svelte 5 + Tailwind CSS 4

**Working directly in demo:**

```bash
cd demo
npm install             # Install demo dependencies (separate from root)
npm run dev             # Start dev server
npm run build           # Production build
npm run ts:check        # TypeScript check
npm run all             # format → lint → ts:check → build
```

Note: The demo has its own `node_modules` and uses Zod for some validation examples.

## Documentation Files

- `README.md` - Main documentation: features, API reference, examples, plugin guide
- `FAQ.md` - Frequently asked questions: common patterns, troubleshooting, Zod integration, per-field dirty tracking
- `docs/llms.txt` - LLM-oriented documentation with demo page descriptions and code snippets

## Architecture

### Core Files

- `src/index.ts` - Public exports: `createSvState`, validator builders, plugin types and built-in plugins, types (`Snapshot`, `EffectContext`, `SnapshotFunction`, `SvStateOptions`, `Validator`, `AsyncValidator`, `AsyncValidatorFunction`, `AsyncErrors`, `DirtyFields`, `SvStatePlugin`, `PluginContext`, `PluginStores`, `ChangeEvent`, `ActionEvent`)
- `src/state.svelte.ts` - Main `createSvState<T, V, P>()` function with snapshot/undo system, async validation, and plugin integration
- `src/proxy.ts` - `ChangeProxy` deep reactive proxy implementation
- `src/validators.ts` - Fluent validator builders (string, number, array, date)
- `src/plugin.ts` - Plugin type definitions (`SvStatePlugin`, `PluginContext`, `PluginStores`, `ChangeEvent`, `ActionEvent`)
- `src/plugins/` - Built-in plugins: `persistPlugin`, `autosavePlugin`, `devtoolsPlugin`, `historyPlugin`, `syncPlugin`, `undoRedoPlugin`, `analyticsPlugin`

### createSvState Function (src/state.svelte.ts)

The main export creates a validated state object with snapshot/undo support:

```typescript
const { data, execute, state, rollback, rollbackTo, reset, destroy } = createSvState(init, actuators?, options?);
```

**Returns:**

- `data` - Deep reactive proxy around the state object (methods on the object are preserved and callable)
- `execute(params)` - Async function to run the configured action
- `rollback(steps?)` - Undo N steps (default 1), restores state and triggers validation
- `rollbackTo(title)` - Roll back to the last snapshot matching `title`, returns `boolean` (true if found)
- `reset()` - Return to initial snapshot, triggers validation
- `destroy()` - Cleanup function: calls plugin `destroy` hooks in reverse order, cancels async validations
- `state` - Object containing reactive stores:
  - `errors: Readable<V | undefined>` - Validation errors (sync)
  - `hasErrors: Readable<boolean>` - Whether any sync validation errors exist
  - `isDirty: Readable<boolean>` - Whether state has been modified (derived from `isDirtyByField`)
  - `isDirtyByField: Readable<DirtyFields>` - Per-field dirty tracking; keys are dot-notation property paths. When a nested field changes, all parent paths are also marked dirty (e.g., changing `customer.address.street` marks `customer.address` and `customer` as dirty). Cleared on `reset()`, `rollback()`, and successful action (respecting `resetDirtyOnAction`).
  - `actionInProgress: Readable<boolean>` - Action execution status
  - `actionError: Readable<Error | undefined>` - Last action error
  - `snapshots: Readable<Snapshot<T>[]>` - Snapshot history for undo

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BCsabaEngine/svstate](https://github.com/BCsabaEngine/svstate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
