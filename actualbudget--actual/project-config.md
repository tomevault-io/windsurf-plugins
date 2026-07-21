---
trigger: always_on
description: This guide provides comprehensive information for AI agents (like Cursor) working with the Actual Budget codebase.
---

# AGENTS.md - Guide for AI Agents Working with Actual Budget

This guide provides comprehensive information for AI agents (like Cursor) working with the Actual Budget codebase.

## Project Overview

**Actual Budget** is a local-first personal finance tool written in TypeScript/JavaScript. It's 100% free and open-source with synchronization capabilities across devices.

- **Repository**: https://github.com/actualbudget/actual
- **Community Docs**: Documentation is part of the monorepo at `packages/docs/`. Published at https://actualbudget.org/docs
- **License**: MIT
- **Primary Language**: TypeScript (with React)
- **Build System**: Yarn 4 workspaces (monorepo)

## Quick Start Commands

### Essential Commands (Run from Root)

```bash
# Type checking (ALWAYS run before committing)
yarn typecheck

# Linting and formatting (with auto-fix)
yarn lint:fix

# Run all tests
yarn test

# Start development server (browser)
yarn start

# Start with sync server
yarn start:server-dev

# Start desktop app development
yarn start:desktop
```

### Important Rules

- **ALWAYS run yarn commands from the root directory** - never run them in child workspaces
- Use `yarn workspace <workspace-name> run <command>` for workspace-specific tasks
- Tests run once and exit by default (using `vitest --run`)

### ⚠️ PR titles must start with `[AI]`

Every pull request title must be prefixed with `[AI]` — you have to apply it
yourself. See [PR and Commit Rules](.github/agents/pr-and-commit-rules.md).

### Task Orchestration with Lage

The project uses **[lage](https://microsoft.github.io/lage/)** (a task runner for JavaScript monorepos) to efficiently run tests and other tasks across multiple workspaces:

- **Parallel execution**: Runs tests in parallel across workspaces for faster feedback
- **Smart caching**: Caches test results to skip unchanged packages (cached in `.lage/` directory)
- **Dependency awareness**: Understands workspace dependencies and execution order
- **Continues on error**: Uses `--continue` flag to run all packages even if one fails

**Lage Commands:**

```bash
# Run all tests across all packages
yarn test                    # Equivalent to: lage test --continue

# Run tests without cache (for debugging/CI)
yarn test:debug              # Equivalent to: lage test --no-cache --continue
```

Configuration is in `lage.config.js` at the project root.

## Architecture & Package Structure

### Core Packages

#### 1. **loot-core** (`packages/loot-core/`)

The core application logic that runs on any platform.

- Business logic, database operations, and calculations
- Platform-agnostic code
- Exports for both browser and node environments
- Test commands:

  ```bash
  # Run all loot-core tests
  yarn workspace @actual-app/core run test

  # Or run tests across all packages using lage
  yarn test
  ```

#### 2. **desktop-client** (`packages/desktop-client/` - aliased as `@actual-app/web`)

The React-based UI for web and desktop.

- React components using functional programming patterns
- E2E tests using Playwright
- Vite for bundling
- Commands:

  ```bash
  # Development
  yarn workspace @actual-app/web start:browser

  # Build
  yarn workspace @actual-app/web build

  # E2E tests
  yarn workspace @actual-app/web e2e

  # Visual regression tests
  yarn workspace @actual-app/web vrt
  ```

#### 3. **desktop-electron** (`packages/desktop-electron/`)

Electron wrapper for the desktop application.

- Window management and native OS integration
- E2E tests for Electron-specific features

#### 4. **api** (`packages/api/` - aliased as `@actual-app/api`)

Public API for programmatic access to Actual.

- Node.js API
- Designed for integrations and automation
- Commands:

  ```bash
  # Build
  yarn workspace @actual-app/api build

  # Run tests
  yarn workspace @actual-app/api test

  # Or use lage to run all tests
  yarn test
  ```

#### 5. **sync-server** (`packages/sync-server/` - aliased as `@actual-app/sync-server`)

Synchronization server for multi-device support.

- Express-based server
- Currently transitioning to TypeScript (mostly JavaScript)
- Commands:
  ```bash
  yarn workspace @actual-app/sync-server start
  ```

#### 6. **component-library** (`packages/component-library/` - aliased as `@actual-app/components`)

Reusable React UI components.

- Shared components like Button, Input, Menu, etc.
- Theme system and design tokens
- Icons (375+ icons in SVG/TSX format)

#### 7. **crdt** (`packages/crdt/` - aliased as `@actual-app/crdt`)

CRDT (Conflict-free Replicated Data Type) implementation for data synchronization.

- Protocol buffers for serialization
- Core sync logic

#### 8. **plugins-service** (`packages/plugins-service/`)

Service for handling plugins/extensions.

#### 9. **eslint-plugin-actual** (`packages/eslint-plugin-actual/`)

Custom ESLint rules specific to Actual.

- `no-untranslated-strings`: Enforces i18n usage
- `prefer-trans-over-t`: Prefers Trans component over t() function
- `prefer-logger-over-console`: Enforces using logger instead of console in `packages/loot-core/`
- `typography`: Typography rules
- `prefer-if-statement`: Prefers explicit if statements

#### 10. **docs** (`packages/docs/`)

Documentation website built with Docusaurus.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [actualbudget/actual](https://github.com/actualbudget/actual) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
