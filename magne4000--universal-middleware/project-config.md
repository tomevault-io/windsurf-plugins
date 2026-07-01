---
trigger: always_on
description: This repository provides a framework for writing server middlewares and handlers once and deploying them across multiple server frameworks (Express, Hono, Fastify, h3, Hattip, Elysia, srvx, Vercel, Cloudflare, Webroute). It enables library authors to write framework-agnostic middleware that can be adapted to any supported platform.
---

# Agent Instructions for universal-middleware

## Repository Overview

This repository provides a framework for writing server middlewares and handlers once and deploying them across multiple server frameworks (Express, Hono, Fastify, h3, Hattip, Elysia, srvx, Vercel, Cloudflare, Webroute). It enables library authors to write framework-agnostic middleware that can be adapted to any supported platform.

**Project Type:** TypeScript monorepo using pnpm workspaces and Turbo for build orchestration  
**Primary Language:** TypeScript (ES Modules)  
**Package Manager:** pnpm 11.5.2 (required)  
**Node Version:** Node.js 22.0.0 or higher (tested on 22, 24)  
**Repository Size:** 15 packages + documentation + examples

## Critical Setup Requirements

### Prerequisites
1. **ALWAYS install pnpm first if not available:** `npm install -g pnpm@11.5.2`
2. **Node version:** Must be 22.0.0 or higher (check with `node --version`)
3. **ALWAYS run `pnpm install` before any other command** - dependencies must be installed fresh

### Initial Setup (Run in Order)
```bash
# 1. Install pnpm globally if needed
npm install -g pnpm@11.5.2

# 2. Install all dependencies (REQUIRED - takes ~20-30 seconds, downloads ~174 MB for playwright)
pnpm install

# 3. Build all packages (REQUIRED before testing/linting - takes ~60 seconds)
pnpm run build
```

**NOTE:** The build step is mandatory before running tests, linting, or typechecking due to turbo.json dependencies.

## Build, Test, and Validation Commands

### Building
```bash
# Build all packages (ALWAYS run this first after install)
pnpm run build
# Takes: ~55-60 seconds on first run, faster with cache
# Builds 18 packages in topological order using Turbo
```

### Linting
```bash
# Run linter (uses Biome)
pnpm run lint
# Takes: ~5 seconds
# Must run AFTER build due to turbo.json dependencies
# Config: biome.json (semicolons: always, indentWidth: 2, lineWidth: 120)
```

### Type Checking
```bash
# Run type checking across all packages
pnpm run test:typecheck
# Takes: ~45 seconds
# Must run AFTER build due to turbo.json dependencies
# Each package has its own tsconfig.json
```

### Testing
```bash
# Run all tests (includes vitest unit tests + integration tests)
pnpm run test
# Takes: Several minutes (includes starting test servers)
# Must run AFTER build due to turbo.json dependencies
# Note: Some tests require Bun and Deno which may not be in all environments
```

### Documentation Build
```bash
# Build documentation site
cd docs && pnpm run build-doc
# Takes: ~50-60 seconds
# Uses VitePress
# Warning: Large chunks (>500kB) are expected - this is normal
```

### Formatting
```bash
# Format code (auto-fixes)
pnpm run format
# Uses Biome formatter with 2-space indentation
```

## Project Structure

### Root Directory Files
- `package.json` - Workspace root with scripts for build/test/lint/release
- `pnpm-workspace.yaml` - Defines workspace packages, shared catalog dependencies, and pnpm settings (`autoInstallPeers: false`, `linkWorkspacePackages: deep`)
- `turbo.json` - Build orchestration config (defines task dependencies)
- `biome.json` - Linter and formatter configuration
- `tsconfig.json` - Base TypeScript configuration (strict mode, ES2022)
- `vitest.workspace.ts` - Test configuration workspace

### Package Structure
```
packages/
├── core/                    # Core utilities and types (@universal-middleware/core)
├── adapter-*/              # Framework adapters (express, hono, fastify, h3, etc.)
│   ├── src/               # TypeScript source
│   ├── test/              # Vitest tests
│   ├── tsup.config.ts     # Build configuration
│   └── vitest.config.ts   # Test configuration
├── compress/              # Compression middleware
├── sirv/                  # Static file serving middleware
├── tests/                 # Shared test utilities
└── universal-middleware/  # Main package with bundler plugins

examples/
└── tool/                  # Example middleware implementations

tests-examples/
└── tests-tool/           # Integration test suites

docs/                     # VitePress documentation site
├── .vitepress/          # VitePress configuration
├── guide/               # User guides
├── reference/           # API reference
├── recipes/             # Code examples
└── package.json         # Has build-doc script
```

### Key Configuration Files

**Build Configuration:**
- Each package uses `tsup` for building (config in `tsup.config.ts`)
- Target: ES2022 for adapters, Node 20 for core/express/fastify
- Output: ESM format to `dist/` directory
- Type definitions generated automatically

**TypeScript:**
- Base config: `tsconfig.json` (strict mode, ESNext modules, bundler resolution)
- Each package extends base config with `{ "extends": "../../tsconfig.json" }`
- No emit from tsconfig - build handled by tsup

**Linting (Biome):**
- Config: `biome.json`
- Rules: All recommended rules enabled
- Formatter: 2-space indent, semicolons always, 120 char line width

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [magne4000/universal-middleware](https://github.com/magne4000/universal-middleware) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
