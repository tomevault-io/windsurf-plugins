---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

web-solc is a monorepo that provides Solidity compilation capabilities for web environments. It consists of:

- **web-solc**: Core package for browser-compatible Solidity compilation using Web Workers
- **@web-solc/react**: React bindings with hooks and context providers
- **example**: Vite-based demo application

## Key Commands

### Development

```bash
# Install dependencies and build all packages
yarn install

# Start development mode (watches all packages)
yarn start

# Build specific package
cd packages/web-solc && yarn prepare
cd packages/react && yarn prepare

# Run the example app only
cd packages/example && yarn dev
```

### Building

```bash
# Build all packages (runs automatically on install)
yarn postinstall

# Build individual packages
cd packages/web-solc && yarn prepare
cd packages/react && yarn prepare
cd packages/example && yarn build
```

### Testing

```bash
# Run all tests across the monorepo
yarn test

# Run tests with coverage
yarn test:coverage

# Run browser integration tests
yarn test:integration

# Run tests for specific packages
cd packages/web-solc && yarn test
cd packages/react && yarn test

# Run tests in watch mode
yarn test:watch

# Run tests with UI
cd packages/web-solc && yarn test:ui

# Run compatibility tests
yarn test:compat:download  # Download all Solidity versions
yarn test:compat           # Run all compatibility tests
yarn test:compat:report    # Generate compatibility report
```

#### Compatibility Testing

The project includes comprehensive Solidity version compatibility testing:

```bash
cd packages/web-solc

# Download all Solidity compiler versions (to vendor/ directory)
yarn test:compat:download

# Run integration tests with representative versions (fast, for local development)
yarn test:integration

# Run ALL version compatibility tests (slower, used in CI)
yarn test:compat

# Generate compatibility report and badge data
yarn test:compat:report
```

**Test Optimization**: By default, integration tests only run against representative versions (latest of each minor version + known edge cases). Set `TEST_ALL_VERSIONS=true` to test all 113+ versions.

**Solidity Files**: Downloaded soljson files are stored in `packages/web-solc/vendor/` which is gitignored to keep the repository size manageable.

**Compatibility Report**: The `COMPATIBILITY.md` and `compatibility-badge.json` files are tracked in git and should be updated when compatibility changes:

1. Run `yarn test:compat:download` to get all compiler versions
2. Run `yarn test:compat:report` to generate the report
3. Commit both files with your changes
4. CI will validate these files are up-to-date on PRs

The project uses **Vitest** for unit testing and **Playwright** for browser integration tests. All packages have comprehensive test coverage including:

- Unit tests for all core modules (browser, node, common)
- React component and hook tests with proper mocking
- Browser integration tests for real compilation scenarios
- Compatibility tests across all Solidity versions

### Compatibility Testing

The repository includes a comprehensive compatibility testing system:

- **COMPATIBILITY.md**: Generated report of all tested Solidity versions (in repository root)
- **compatibility-badge.json**: Badge data for README display (in repository root)
- **bin/generate-compatibility-report.ts**: Script to run all tests and generate report
- **Fast integration tests**: 8 representative versions in `tests/integration/`
- **Full compatibility tests**: 113+ versions in `tests/compatibility/`
- Separate test environments to capture browser vs Node.js differences
- Compatibility tests for all stable Solidity versions (0.4.11+)

## Recent API Changes

### web-solc (core package)

- **Renamed**: `soljsonText` parameter renamed to `soljson` throughout the API
- **New function**: `resolveSolc(versionRange)` - resolves version ranges to exact versions without downloading
- **Updated**: `fetchSolc` now uses `resolveSolc` internally for consistent version resolution

### @web-solc/react

- **Redesigned**: `useWebSolc` hook now works standalone without requiring `WebSolcProvider`
- **Cache changes**: Cache now stores soljson strings instead of WebSolc instances
  - Interface renamed from `CompilerCache` to `SoljsonCache`
  - Simpler implementation for third-party caching strategies
  - Hook handles all compiler lifecycle internally
- **Provider optional**: `WebSolcProvider` is now optional and primarily for caching across components

## Architecture

### Core Design Principles

1. **Dual Environment Support**: The core web-solc package provides different implementations for browser (Web Workers) and Node.js environments
2. **Version Resolution**: Uses semver to fetch compatible Solidity compiler versions from binaries.soliditylang.org
3. **Worker Isolation**: Browser compilation runs in Web Workers to prevent blocking the main thread
4. **Type Safety**: Full TypeScript with strict mode enabled

### Key Implementation Details

**Browser Implementation** (`packages/web-solc/src/browser.ts`):

- Creates Web Workers using Blob URLs with embedded worker code

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gnidan/web-solc](https://github.com/gnidan/web-solc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
