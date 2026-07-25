---
trigger: always_on
description: This file provides quick-reference guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides quick-reference guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **For detailed information**, see [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md) - the comprehensive development guide.

## Project Overview

Deneb is a custom Power BI visual that enables declarative visualization using Vega and Vega-Lite languages. It's an npm workspaces monorepo using Turbo for orchestration and a custom Webpack 5 build system.

**Key Technologies:** React 19, Vega 6.2, Vega-Lite 6.4, Monaco Editor, Fluent UI, Zustand, TypeScript 5.6, Webpack 5, Power BI Visuals API 5.11

## Common Commands

### Development

```bash
npm run dev                          # Start all package watchers + webpack dev server (https://localhost:8080)
npm run webpack:build                # One-off dev build without server
```

### Testing & Linting

```bash
npm run test                         # Run all tests across packages (Vitest)
npm run test:watch                   # Watch mode for tests
npm run eslint                       # Lint all packages + root
npm run prettier-check               # Check code formatting
npm run prettier-format              # Auto-format code
```

### Production Packaging

```bash
npm run package                      # Certified build for AppSource (validates config first)
npm run package-standalone           # Developer build with external URIs enabled
npm run package-alpha                # Alpha channel build
npm run package-beta                 # Beta channel build
npm run webpack:analyze              # Generate bundle size analysis (webpack.statistics.html)
```

### Utilities

```bash
npm run validate-config-for-commit   # Validate .env flags for certification (critical before packaging)
npm run clean                        # Clean all build artifacts
npm run sync-packages                # Sync package versions across monorepo
```

### First-Time Setup

```bash
npm install       # Install dependencies
npm run dev       # Start development (clears .tmp/, builds packages, primes assets, starts server)
```

> **Details**: See [First-time setup](doc/DEVELOPMENT.md#first-time-setup) in DEVELOPMENT.md

## High-Level Architecture

### Monorepo Structure

**Root Package (`@deneb-viz/deneb`):** Main Power BI custom visual

- Entry: [src/index.ts](src/index.ts) exports `Deneb` class implementing `IVisual`
- Visual GUID: `deneb7E15AEF80B9E4D4F8E12924291ECE89A`
- Integrates all workspace packages into final `.pbiviz` bundle

**Workspace Packages (`packages/`):**

- **app-core** - Core UI application (React components, Monaco editor, state management)
- **vega-runtime** - Vega/Vega-Lite runtime integration, spec processing, and compilation
- **vega-react** - React hooks and context for Vega embedding (useVegaEmbed, VegaViewProvider)
- **powerbi-compat** - Power BI API compatibility layer (**SINGLETON** - see below)
- **data-core** - Dataset field management and value processing; includes the support field processing engine (types, plan builder, row builder, default provider)
- **json-processing** - JSON spec processing and field tracking
- **configuration** - Configuration and feature flags
- **template-usermeta** - Template metadata handling
- **utils** - Shared utilities (logging, crypto, etc.)
- **eslint-config**, **typescript-config** - Shared tooling configs

**Apps (`apps/`):**

- **web-client-sample** - Vite-based web integration sample

**Documented Solutions (`docs/solutions/`):** Past problems diagnosed and solved, organized by category with YAML frontmatter (`module`, `tags`, `problem_type`). Relevant when debugging or implementing in documented areas.

### app-core Layering

`@deneb-viz/app-core` follows a strict layered model: `app/ → features/ → components/ → lib/ → state/context/i18n/catalog`. Cross-feature imports and upward imports are rejected by `eslint-plugin-boundaries` (configured in [packages/app-core/eslint.config.js](packages/app-core/eslint.config.js)) and gated by a vitest canary at [packages/app-core/src/**tests**/architecture-boundaries.test.ts](packages/app-core/src/__tests__/architecture-boundaries.test.ts). Full details — including the per-layer dependency matrix and a decision guide for where new code goes — live in [packages/app-core/ARCHITECTURE.md](packages/app-core/ARCHITECTURE.md).

### Critical: Singleton Package Pattern

`@deneb-viz/powerbi-compat` MUST remain a singleton to maintain shared runtime state:

- Packages consuming it declare it as `peerDependency` (not `dependency`)
- Mark as `external` in tsup configs to prevent bundling
- Root visual provides the single runtime instance
- Uses TypeScript compiler (tsc) instead of tsup to inline const enums from `powerbi-visuals-api`

**When adding dependencies on `@deneb-viz/powerbi-compat`:**

1. Add to `peerDependencies` in consuming package's package.json
2. Add to `external` array in consuming package's tsup.config.ts
3. Never bundle it - let the root visual provide the singleton instance

### Compilation Architecture

The spec compilation flow in `@deneb-viz/vega-runtime` and `@deneb-viz/app-core`:

```
User Spec (JSONC) → parseSpec() → Signal migration (pbiContainer→denebContainer)
    ↓

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deneb-viz/deneb](https://github.com/deneb-viz/deneb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
