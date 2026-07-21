---
trigger: always_on
description: This file provides guidance to AI coding agents working in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents working in this repository.

## Project Overview

This is the **Cloudflare Workers SDK** monorepo containing tools and libraries for developing, testing, and deploying applications on Cloudflare. The main components are Wrangler (CLI), Miniflare (local dev simulator), and Create Cloudflare (project scaffolding).

## Development Commands

**Package Management:**

- Use `pnpm` - never use npm or yarn
- `pnpm install` - Install dependencies for all packages
- `pnpm build` - Build all packages (uses Turbo for caching)

**Testing:**

- `pnpm test:ci` - Run tests in CI mode
- `pnpm test:e2e` - Run end-to-end tests (requires Cloudflare credentials)
- `pnpm test -F <package> "pattern"` - Run a single test by name pattern

**Code Quality:**

- `pnpm check` - Run all checks (lint, type, format)
- `pnpm fix` - Auto-fix linting issues and format code

**Working with Specific Packages:**

- `pnpm run build --filter <package-name>` - Build specific package
- `pnpm run test:ci --filter <package-name>` - Test specific package
- `pnpm --filter <package> test:watch` - Watch mode for a specific package

## Architecture Overview

**Core Tools:**

- `packages/wrangler/` - Main CLI tool for Workers development and deployment
- `packages/miniflare/` - Local development simulator powered by workerd runtime
- `packages/create-cloudflare/` - Project scaffolding CLI (C3)
- `packages/vite-plugin-cloudflare/` - Vite plugin for Cloudflare Workers

**Development & Testing:**

- `packages/vitest-pool-workers/` - Vitest integration for testing Workers in actual runtime
- `packages/chrome-devtools-patches/` - Modified Chrome DevTools for Workers debugging

**Shared Libraries:**

- `packages/pages-shared/` - Code shared between Wrangler and Cloudflare Pages
- `packages/workers-shared/` - Code shared between Wrangler and Workers Assets
- `packages/workers-utils/` - Utility package for common Worker operations
- `packages/workflows-shared/` - Internal Cloudflare Workflows functionality
- `packages/containers-shared/` - Shared container functionality
- `packages/unenv-preset/` - Cloudflare preset for unenv (Node.js polyfills)
- `packages/cli/` - SDK for building workers-sdk CLIs
- `packages/kv-asset-handler/` - KV-based asset handling for Workers Sites

**Build System:**

- Turbo (turborepo) orchestrates builds across packages
- TypeScript compilation with shared configs in `packages/workers-tsconfig/`
- Shared lint config in `packages/lint-config-shared/`
- Dependency management via pnpm catalog system

## WHERE TO LOOK

| Task                                           | Location                                                              | Notes                                                                                                                                                                        |
| ---------------------------------------------- | --------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Add/modify a CLI command                       | `packages/wrangler/src/`                                              | Commands registered in `src/index.ts` (2k+ line yargs tree)                                                                                                                  |
| Change local dev behavior                      | `packages/miniflare/src/`                                             | `src/index.ts` is the main `Miniflare` class                                                                                                                                 |
| Modify Workers runtime simulation              | `packages/miniflare/src/workers/`                                     | ~30 embedded worker scripts, built via `worker:` virtual imports                                                                                                             |
| Add a test fixture                             | `fixtures/`                                                           | Each fixture is a full workspace member with own `package.json`                                                                                                              |
| Shared config types/validation                 | `packages/workers-utils/src/config/`                                  | `validation.ts` is the config normalizer (large file)                                                                                                                        |
| Test helpers (runInTempDir, seed, mockConsole) | `packages/workers-utils/src/test-helpers/`                            | Shared across wrangler, miniflare, others                                                                                                                                    |
| Cloudflare API mocks for tests                 | `packages/wrangler/src/__tests__/helpers/msw/`                        | MSW handlers per API domain                                                                                                                                                  |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudflare/workers-sdk](https://github.com/cloudflare/workers-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
