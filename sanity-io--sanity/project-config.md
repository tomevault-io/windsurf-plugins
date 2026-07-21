---
trigger: always_on
description: This document helps AI agents work successfully with the Sanity monorepo.
---

# AGENTS.md - AI Agent Guidelines for Sanity Monorepo

This document helps AI agents work successfully with the Sanity monorepo.

## Prerequisites

- **Node.js**: v24 or latest LTS
- **Package Manager**: pnpm v10+ (exact version managed via `packageManager` field in package.json)

## Quick Reference

```bash
# Install dependencies (pnpm ONLY - enforced)
pnpm install

# Build all packages (required before testing)
pnpm build

# Run dev studio (requires auth, see below)
pnpm dev

# Format code (MUST pass CI)
pnpm chore:format:fix

# Fix all lint issues (MUST pass CI)
pnpm lint:fix

# Run tests
pnpm test

# Update snapshots if tests fail due to expected changes
pnpm test -- -u

# Type check
pnpm check:types
```

## CI Checks - What Must Pass

These checks run on every PR and **must pass**:

| Check            | Command               | Notes                                                                                                                                                            |
| ---------------- | --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Format**       | `pnpm check:format`   | Uses oxfmt. Fix with `pnpm chore:format:fix`                                                                                                                     |
| **Oxlint**       | `pnpm check:oxlint`   | Rust linter (type-aware via tsgolint) plus ESLint plugins loaded as oxlint jsPlugins. Fix with `pnpm chore:oxlint:fix`                                           |
| **Type Check**   | `pnpm check:types`    | TypeScript via tsgo + turbo                                                                                                                                      |
| **Unit Tests**   | `pnpm test`           | Vitest, sharded in CI                                                                                                                                            |
| **Export Tests** | `pnpm test:exports`   | Ensures ESM/CJS/DTS work                                                                                                                                         |
| **Dep Check**    | `pnpm depcheck`       | Finds unused/missing deps                                                                                                                                        |
| **Zizmor**       | `pnpm lint:workflows` | Audits `.github/workflows/` for security issues. Fails CI on high-severity findings. Local run needs [`zizmor`](https://docs.zizmor.sh/installation/) on `PATH`. |
| **PR Title**     | Conventional commits  | e.g., `feat(scope): description`                                                                                                                                 |

### Before Committing

Run these commands to avoid CI failures:

```bash
# Fix all formatting and lint issues
pnpm lint:fix

# Verify tests pass (build first if needed)
pnpm build && pnpm test
```

If tests fail due to **expected snapshot changes**, update them:

```bash
pnpm test -- -u
```

Snapshot files are located in `__snapshots__` directories alongside test files.

## Project Structure

```
sanity/
├── packages/
│   ├── sanity/           # Main Sanity studio package
│   ├── @sanity/          # Scoped packages (cli, types, schema, etc.)
│   └── @repo/            # Internal tooling (test-config, tsconfig, etc.)
├── dev/                  # Development studios for testing
│   └── test-studio/      # Primary dev studio (pnpm dev runs this)
├── e2e/                  # End-to-end Playwright tests
├── perf/                 # Performance testing
└── examples/             # Example studios
```

### Key Packages

- **`packages/sanity`** - Core studio package with all UI components
- **`packages/@sanity/types`** - TypeScript type definitions
- **`packages/@sanity/schema`** - Schema compilation
- **`packages/@sanity/mutator`** - Document mutation logic

## Build System

- **Package Manager**: pnpm (version 10.x, enforced via `preinstall`)
- **Build Orchestration**: Turbo (caches builds)
- **Versioning**: Lerna-lite with conventional commits

### Build Commands

```bash
pnpm build              # Build all packages
pnpm watch              # Watch mode for development
```

### Running the Dev Studio

```bash
pnpm dev                # Starts dev studio at http://localhost:3333
```

**Note:** The dev studio requires Sanity user authentication in the browser. It's a Vite application that communicates with Sanity API endpoints, so you'll need to log in with a Sanity account when you access `http://localhost:3333` to use the studio.

## Local Development

This section clarifies what requires authentication and what doesn't—critical for AI agents to avoid getting stuck on auth flows.

### Running Tests (No Auth Required)

Unit tests run in jsdom with mocks and **do not require any authentication**:

```bash
# Build first (required), then run all tests
pnpm build && pnpm test

# Run a single test file (IMPORTANT: use vitest directly with --project to avoid running all tests)
pnpm vitest run --project=sanity packages/sanity/src/core/hooks/useClient.test.ts


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sanity-io/sanity](https://github.com/sanity-io/sanity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
