---
trigger: always_on
description: Welcome to the CSpell monorepo! This document provides essential information for efficiently working with this codebase.
---

# CSpell Monorepo - Copilot Coding Agent Instructions

Welcome to the CSpell monorepo! This document provides essential information for efficiently working with this codebase.

## Repository Overview

CSpell is a spell checker for code that supports multiple programming languages and is available as a command-line tool, library, and ESLint plugin. This is a monorepo containing ~30+ packages managed with **pnpm workspaces** and **Lerna Lite**.

### Key Packages

- **cspell** - Main CLI application
- **cspell-lib** - Core spelling engine and API
- **cspell-types** - TypeScript types and JSON schema
- **cspell-tools** - Dictionary compilation tools
- **cspell-trie-lib** - Trie data structure for word storage
- **cspell-eslint-plugin** - ESLint integration
- **cspell-bundled-dicts** - Collection of bundled dictionaries

## Development Environment Setup

### Prerequisites

- **Node.js:** 22.18.0+ (CI tests on 22.x, 24.x & 26.x)
- **Package Manager:** pnpm 10.27.0+ (enforced via `packageManager` field)
- **Corepack:** Enable with `corepack enable` (required for correct pnpm version)

### Initial Setup

```bash
# Enable corepack (ensures correct pnpm version)
corepack enable

# Install dependencies
pnpm install

# Build all packages
pnpm run build

# Run tests
pnpm test

# Or do all at once
pnpm ibt  # Install → Build → Test
```

## Build System

### Build Tools

- **TypeScript 5.9.3** - Primary language
- **tsdown** - Fast TypeScript bundler (used by most packages)
- **rollup** - Module bundler (for some packages)
- **vitest** - Test runner (primary)
- **jest** - Test runner (legacy, being phased out)

### Build Commands

```bash
# Build all packages (includes test-packages)
pnpm run build:all

# Build only production packages (excludes test-packages)
pnpm run build:prod

# Build specific package
cd packages/cspell && pnpm build

# Clean build artifacts
pnpm run clean

# Clean and rebuild everything
pnpm run clean-build
```

### Build Configuration

- **tsconfig.json** - Root TypeScript config (extends `@tsconfig/node22`)
- **tsdown.config.ts** - Package-level bundler config (ESM + CJS outputs)
- **vitest.config.mjs** - Test configuration
- **eslint.config.mjs** - ESLint configuration

### Important Build Notes

- **Workspace concurrency:** Build uses `--workspace-concurrency=2` to prevent race conditions
- **Dual format output:** Most packages output both ESM (dist/esm) and CJS (dist/cjs)
- **Declaration files:** Generated with `isolatedDeclarations: true`
- **Source maps:** Enabled for all builds
- **Build validation:** CI runs `git diff --exit-code` after builds to ensure clean state

## Testing

### Test Commands

```bash
# Run all tests
pnpm test

# Run tests in watch mode
pnpm run test-watch

# Run specific package tests
cd packages/cspell && pnpm test

# Update test snapshots
pnpm run test:update-snapshots

# Run performance tests
pnpm run test:perf

# Run integration tests
pnpm run test-integrations

# Generate coverage
pnpm run coverage
# Open coverage/index.html to view report
```

### Test Frameworks

- **vitest** - Primary test runner (modern, fast)
- **jest** - Legacy test runner (some packages still use it)

### Test Patterns

- Test files: `*.test.ts`, `*.spec.ts`, `*.perf.ts`
- Snapshots: `__snapshots__/` directories
- Fixtures: `fixtures/` directories in packages
- Integration tests: `integration-tests/` directory (tests against real repos)

## Linting and Code Quality

### Linting Commands

```bash
# Run ESLint and Prettier
pnpm run lint

# Lint without cache (slower but comprehensive)
pnpm run eslint-fix-no-cache

# Check code without fixing
pnpm run lint-ci

# Format markdown in website
pnpm run lint-docs
```

### Code Style

- **ESLint:** Uses flat config (eslint.config.mjs) with TypeScript, Node, and Unicorn plugins
- **Prettier:** 120 character line width, single quotes, trailing commas (except JSON/MD)
- **EditorConfig:** 4 spaces for most files, 2 spaces for YAML/JSON/MD
- **Line endings:** LF (enforced via .editorconfig and TypeScript config)

### Pre-commit Checks

- Spelling: `pnpm run check-spelling` (uses cspell on the codebase itself)
- Git validation: Ensure clean state after builds

## Monorepo Structure

### Workspace Configuration

The repository uses **pnpm workspaces** defined in `pnpm-workspace.yaml`:

```yaml
packages:
  - 'packages/*'
  - 'test-packages/*/*'
  - 'integration-tests'
  - 'website'
  - 'tools/*'
  - 'rfc/*'
  # ... and more
```

### Internal Dependencies

- Use `workspace:*` notation for internal package dependencies
- Example: `"cspell-lib": "workspace:*"`
- pnpm automatically resolves to local packages during development

### Package Structure

```
packages/
├── cspell/              # Main CLI package
├── cspell-lib/          # Core library
├── cspell-types/        # Types and schemas
├── cspell-tools/        # Dictionary tools
└── ...
test-packages/           # Integration/format tests
integration-tests/       # Tests against real repositories
website/                 # Documentation site (Docusaurus)
tools/                   # Build and automation tools
scripts/                 # Helper scripts
```

## Common Workflows

### Adding a New Package

1. Create directory in `packages/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [streetsidesoftware/cspell](https://github.com/streetsidesoftware/cspell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
