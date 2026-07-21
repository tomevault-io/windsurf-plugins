---
trigger: always_on
description: > **Note:** `CLAUDE.md` is a symlink to `AGENTS.md`. They are the same file.
---

# Next.js Development Guide

> **Note:** `CLAUDE.md` is a symlink to `AGENTS.md`. They are the same file.

## Codebase structure

### Monorepo Overview

This is a pnpm monorepo containing the Next.js framework and related packages.

```
next.js/
├── packages/           # Published npm packages
├── turbopack/          # Turbopack bundler (Rust) - git subtree
├── crates/             # Rust crates for Next.js SWC bindings
├── test/               # All test suites
├── examples/           # Example Next.js applications
├── docs/               # Documentation
└── scripts/            # Build and maintenance scripts
```

### Core Package: `packages/next`

The main Next.js framework lives in `packages/next/`. This is what gets published as the `next` npm package.

**Source code** is in `packages/next/src/`.

**Key entry points:**

- Dev server: `src/cli/next-dev.ts` → `src/server/dev/next-dev-server.ts`
- Production server: `src/cli/next-start.ts` → `src/server/next-server.ts`
- Build: `src/cli/next-build.ts` → `src/build/index.ts`

**Compiled output** goes to `packages/next/dist/` (mirrors src/ structure).

### Other Important Packages

- `packages/create-next-app/` - The `create-next-app` CLI tool
- `packages/next-swc/` - Native Rust bindings (SWC transforms)
- `packages/eslint-plugin-next/` - ESLint rules for Next.js
- `packages/font/` - `next/font` implementation
- `packages/third-parties/` - Third-party script integrations

### README files

Before editing or creating files in any subdirectory (e.g., `packages/*`, `crates/*`), read all `README.md` files in the directory path from the repo root up to and including the target file's directory. This helps identify any local patterns, conventions, and documentation.

**Example:** Before editing `turbopack/crates/turbopack-ecmascript-runtime/js/src/nodejs/runtime/runtime-base.ts`, read:

- `turbopack/README.md` (if exists)
- `turbopack/crates/README.md` (if exists)
- `turbopack/crates/turbopack-ecmascript-runtime/README.md` (if exists)
- `turbopack/crates/turbopack-ecmascript-runtime/js/README.md` (if exists - closest to target file)

## Build Commands

```bash
# Build the Next.js package
pnpm --filter=next build

# Build all JS code
pnpm build

# Build all JS and Rust code
pnpm build-all

# Run specific task
pnpm --filter=next exec taskr <task>
```

## Fast Local Development

For iterative development, default to watch mode plus the explicit test script that matches the mode and bundler being verified.

**Default agent rule:** If you are changing Next.js source or integration tests, start `pnpm --filter=next dev` in a separate terminal session before making edits (unless it is already running). If you skip this, explicitly state why (for example: docs-only, read-only investigation, or CI-only analysis).

**1. Start watch build in background:**

```bash
# Auto-rebuilds on file changes (~1-2s per change vs ~60s full build)
# Keep this running while you iterate on code
pnpm --filter=next dev
```

**2. Run focused tests with the matching mode script:**

```bash
# Development mode with Turbopack
pnpm test-dev-turbo test/path/to/test.ts

# Development mode with Webpack
pnpm test-dev-webpack test/path/to/test.ts

# Production build+start with Turbopack
pnpm test-start-turbo test/path/to/test.ts

# Production build+start with Webpack
pnpm test-start-webpack test/path/to/test.ts
```

**3. When done, kill the background watch process (if you started it).**

**For type errors only:** Use `pnpm --filter=next types` (~10s) instead of `pnpm --filter=next build` (~60s).

After the workspace is bootstrapped, prefer `pnpm --filter=next build` when edits are limited to core Next.js files. Use full `pnpm build-all` for branch switches/bootstrap, before CI push, or when changes span multiple packages.

**Always run a full bootstrap build after switching branches:**

```bash
git checkout <branch>
pnpm build-all   # Sets up outputs for dependent packages (Turborepo dedupes if unchanged)
```

## Bundler Selection

Turbopack is the default bundler for both `next dev` and `next build`. To force webpack:

```bash
next build --webpack        # Production build with webpack
next dev --webpack          # Dev server with webpack
```

There is no `--no-turbopack` flag.

## Testing

```bash
# Run specific test file (development mode with Turbopack)
pnpm test-dev-turbo test/path/to/test.test.ts

# Run tests matching pattern
pnpm test-dev-turbo -t "pattern"

# Run development tests
pnpm test-dev-turbo test/development/
```

**Test commands by mode:**

- `pnpm test-dev-turbo` - Development mode with Turbopack (default)
- `pnpm test-dev-webpack` - Development mode with Webpack
- `pnpm test-start-turbo` - Production build+start with Turbopack
- `pnpm test-start-webpack` - Production build+start with Webpack

**Other test commands:**

- `pnpm test-unit` - Run unit tests only (fast, no browser)
- `pnpm new-test` - Generate a new test file from template (interactive)

**Generate tests non-interactively (for AI agents):**

Generating tests using `pnpm new-test` is mandatory.

```bash
# Use --args for non-interactive mode (forward args to the script using `--`)
# Format: pnpm new-test -- --args <appDir> <name> <type>
# appDir: true/false (is this for app directory?)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vercel/next.js](https://github.com/vercel/next.js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
