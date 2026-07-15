---
trigger: always_on
description: This document provides agents with essential information for working on the snap-7715-permissions monorepo.
---

# AGENTS.md - snap-7715-permissions Project Guide

This document provides agents with essential information for working on the snap-7715-permissions monorepo.

## Project Overview

This is a monorepo implementing ERC-7715 permissions for MetaMask Snaps. It contains:

- **@metamask/permissions-kernel-snap** (`packages/permissions-kernel-snap`) - Kernel snap managing the permissions offer registry
- **@metamask/gator-permissions-snap** (`packages/gator-permissions-snap`) - DeleGator permissions snap that creates delegation accounts
- **@metamask/shared** (`packages/shared`) - Shared utilities, types, constants, and testing utilities across snaps
- **Development/test site** (`packages/site`) - Local testing environment for dApp development

### Architecture Summary

- **Kernel Snap**: Manages a `permissions offer registry` listing all permissions a user is willing to grant via ERC-7715 requests
- **Gator Snap**: Creates DeleGator accounts and enables sites to request ERC-7715 permissions with user review via custom UI dialogs

## Technology Stack

- **Node Version**: 20.x or 22.x (see `.nvmrc`: `^20 || >=22`)
- **Package Manager**: Yarn 4.10.1 with workspaces
- **Language**: TypeScript 5.8.3 (strict mode)
- **Testing**: Jest with @metamask/snaps-jest
- **Linting**: ESLint 9 (flat config - `eslint.config.mjs`)
- **Formatting**: Prettier 3.6.2
- **Runtime**: MetaMask Snaps (Flask >= 12.14.2)

## Package Structure

```
packages/
├── permissions-kernel-snap/  # Kernel snap (port 8081)
├── gator-permissions-snap/   # DeleGator snap (port 8082)
├── shared/                   # Shared utilities and types
└── site/                     # Development testing site (port 8000)
```

## Prerequisites

1. **MetaMask Flask**: >= 12.14.2
2. **Node.js**: 20.x or 22.x (use `nvm use` to switch)
3. **Yarn**: 4.10.1 (managed via `packageManager` field)
4. **Environment variables**: See `.env.example` in each package

## Quick Start

```bash
# Install dependencies and set up snap submodules
yarn prepare:snap

# Build all packages
yarn build

# Start development servers (site on port 8000, snaps on 8081/8082)
yarn start
```

## Build Commands

```bash
# Install dependencies
yarn install

# Full setup with snap submodules
yarn prepare:snap

# Build all packages (parallel, topological)
yarn build

# Build and pack for distribution
yarn build:pack

# Update and validate changelogs
yarn changelog:update
yarn changelog:validate
```

## Development Commands

```bash
# Start all development servers
yarn start
# Access at http://localhost:8000/
# - permissions-kernel-snap: local:http://localhost:8081
# - gator-permissions-snap: local:http://localhost:8082

# Run tests (parallel across workspaces)
yarn test

# Watch mode for tests
yarn test --watch

# Run tests in specific package
yarn workspace @metamask/gator-permissions-snap test

# Linting
yarn lint              # Run all linters
yarn lint:eslint       # ESLint only
yarn lint:fix          # Fix linting issues
yarn lint:misc         # Check markdown, JSON, etc.
```

## Environment Variables

All packages throw build-time errors if required env vars are missing. Check `.env.example` in each package.

### Common Variables

| Variable | Description | Package |
|----------|-------------|---------|
| `SNAP_ENV` | Environment (development/production) | All |
| `KERNEL_SNAP_ID` | Snap ID of permissions kernel snap | gator-snap |
| `STORE_PERMISSIONS_ENABLED` | Feature flag for storage ("true"/"false") | All snaps |

### Package-Specific Setup

Each package requires specific environment variables. See:
- `packages/permissions-kernel-snap/.env.example`
- `packages/gator-permissions-snap/.env.example`
- `packages/site/.env.example`

## Code Style and Standards

### Formatting Requirements

- **Prettier** automatically formats code with:
  - Single quotes (`'`)
  - 2-space indentation
  - Trailing commas throughout
  - Quote props as-needed

All code must be formatted before committing. Run `yarn lint:fix` to auto-fix.

### TypeScript Strictness

The project uses strict TypeScript configuration:

- `strict: true` - Full type checking
- `exactOptionalPropertyTypes: true` - No implicit undefined on optional properties
- `noUncheckedIndexedAccess: true` - Require null checks on indexed access
- Target: ES2020, Module: Node16

### ESLint Configuration

The project uses ESLint 9 with flat config (`eslint.config.mjs`). Key rules:

- No `console.log` in production code
- No unused variables
- No untyped `any` without `@ts-expect-error`
- Imports must be properly sorted
- JSDoc comments required for public APIs (classes and function declarations)
- Only allow throwing Snap SDK error objects

## Testing Standards

- **Test files**: Use `*.test.ts` suffix (not `*.spec.ts`)
- **Structure**: Co-located with source or in `test/` directories
- **Framework**: Jest with @metamask/snaps-jest
- **Pattern**: Arrange-Act-Assert
- **Coverage**: Test happy paths AND error cases
- **Mocking**: Mock external dependencies (no real HTTP calls)

Example:
```typescript
describe('parsePermission', () => {
  it('parses valid permission objects', () => {
    const input = { name: 'test', args: [] };
    const result = parsePermission(input);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MetaMask/snap-7715-permissions](https://github.com/MetaMask/snap-7715-permissions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
