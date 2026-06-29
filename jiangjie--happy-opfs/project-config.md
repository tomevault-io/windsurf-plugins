---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Project Overview

happy-opfs is a browser-compatible file system module based on OPFS (Origin Private File System) that provides a Deno-style API for file operations. The project uses TypeScript and targets ESNext with strict type checking.

**License:** MIT

**Key Dependencies:**
- `@std/path` from JSR - **Important:** Requires `.npmrc` configuration with `@jsr:registry=https://npm.jsr.io`
- `happy-rusty` - Provides Rust-style `Result<T, E>` types for error handling
- `fflate` - For zip/unzip operations (uses `fflate/browser` for browser-optimized build)
- `@happy-ts/fetch-t` - For download/upload operations
- `tiny-future` - For Promise-based future/deferred patterns

### Top-Level Layout

- `src/` - Library source (see detailed tree under *Code Architecture*)
- `tests/` - Vitest + Playwright browser tests and MSW mocks
- `examples/` - Runnable feature demos (served over HTTPS via `vite-plugin-mkcert`)
- `benchmarks/` - Performance benchmarks (served over HTTP on the default Vite port)
- `dist/` - Build output (`main.cjs`, `main.mjs`, `types.d.ts`)
- `docs/` - Generated TypeDoc output (not committed to source; regenerate with `pnpm run docs`)

### Tooling Versions

`package.json` is the source of truth for tool versions. The repo uses `pnpm` (see `pnpm-workspace.yaml` below) and TypeScript. When in doubt, read `package.json` rather than relying on documented versions.

### pnpm Workspace

`pnpm-workspace.yaml` is present and currently only sets `allowBuilds: { msw: true }` to permit MSW's postinstall step. The repo is a single package — do not assume a monorepo layout or add workspace packages without explicit direction.

## Development Commands

### Package Manager
This project uses **pnpm** as the package manager.

### Common Commands
```bash
# Install dependencies
pnpm install

# Type checking
pnpm run check

# Linting
pnpm run lint

# Build (runs prebuild: check types, lint)
pnpm run build

# Generate documentation
pnpm run docs

# Run examples (opens https://localhost:5173)
pnpm run eg
```

### Testing
Tests use **Vitest** with **Playwright** browser automation.

```bash
# Install Playwright browsers (first time setup)
pnpm run playwright:install

# Run all tests
pnpm test

# Run tests in watch mode
pnpm run test:watch

# Run tests with UI
pnpm run test:ui

# Run a specific test file
pnpm exec vitest run tests/core.test.ts

# Run tests matching a pattern
pnpm exec vitest run -t "readFile"
```

**MSW service-worker freshness:** `pnpm test` runs a `pretest` hook (`msw init tests/public`) that regenerates the mock service worker. If you invoke `vitest` directly (bypassing the npm script), the MSW worker may be stale — run `pnpm exec msw init tests/public --save=false` manually, or prefer `pnpm test` / `pnpm run test:watch`. The same applies to benchmarks (`prebench` → `benchmarks/public`).

Tests are located in `tests/` directory. The test environment:
- Uses Playwright's Chromium browser in headless mode
- Automatically configures HTTPS and required COOP/COEP headers
- Runs tests sequentially to avoid OPFS conflicts
- Coverage reports via v8 provider
- Uses MSW (Mock Service Worker) for download/upload API mocking

**Worker helper files for tests:**
- `tests/worker.ts` - Main worker for sync API tests
- `tests/worker-check-connected.ts` - Worker for connection checking tests
- `tests/worker-async-api.ts` - Worker for async API tests

### Mock Server (MSW)
Download and upload tests use MSW instead of external APIs:
- Handlers defined in `tests/mocks/handlers.ts`
- Browser setup in `tests/mocks/browser.ts`
- Service worker is auto-generated via `pretest` script (runs `msw init`)
- Mock endpoints use `https://mock.test` domain

### Code Architecture

#### Module Organization

The project organizes code into three main layers based on execution context and visibility:

> When modifying `src/` structure, keep the tree below in sync — or prefer reading the actual directory and treating this tree as a snapshot. `package.json` and the `src/` tree on disk are the authoritative sources; this document can drift.

```
src/
├── mod.ts                      # Main entry point, exports all public APIs
│
├── shared/                     # Shared utilities (thread-agnostic)
│   ├── mod.ts                 # Aggregates shared modules
│   ├── constants.ts           # Application-wide constants
│   ├── defines.ts             # Shared TypeScript type definitions
│   ├── guards.ts              # Type guard functions (isAbsolutePath, etc.)
│   ├── support.ts             # OPFS feature detection
│   └── internal/              # Internal utilities (@internal tagged)
│       ├── mod.ts             # Aggregates internal modules
│       ├── codec.ts           # UTF-8 encoding/decoding with cached encoders
│       ├── helpers.ts         # Shared helper functions
│       └── validations.ts     # Path/URL validation
│
├── async/                      # Async OPFS operations (main thread)
│   ├── mod.ts                 # Aggregates all async modules
│   ├── core/                  # Core OPFS operations
│   │   ├── mod.ts             # Aggregates core modules

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JiangJie/happy-opfs](https://github.com/JiangJie/happy-opfs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
