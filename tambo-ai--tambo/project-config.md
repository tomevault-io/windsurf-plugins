---
trigger: always_on
description: Detailed guidance for Claude Code agents working with the Tambo AI monorepo.
---

# AGENTS.md

Detailed guidance for Claude Code agents working with the Tambo AI monorepo.

This file provides comprehensive instructions for maintaining code quality, architectural consistency, and project-specific requirements across both the Tambo AI framework and the Tambo Cloud platform.

## 1. Repository Structure

This is a Turborepo monorepo containing both the Tambo AI framework packages and the Tambo Cloud platform.

### Framework Packages (Turborepo root)

- **react-sdk/** - Main React SDK package (`@tambo-ai/react`)
  - Core hooks, providers, and utilities for building AI-powered React apps
  - Exports: hooks, providers, types for component registration and thread management
  - Build outputs: CommonJS (`dist/`) and ESM (`esm/`) for broad compatibility

- **packages/client/** - Framework-agnostic client (`@tambo-ai/client`)
  - Streaming, tool execution, thread management without React dependencies
  - `TamboClient` class with `getState()`/`subscribe()` for framework integration
  - `TamboStream` async iterable for streaming AI responses
  - Used by `@tambo-ai/react` as its core engine; also usable standalone (Node.js, Vue, Svelte, etc.)
  - Build outputs: CommonJS (`dist/`) and ESM (`esm/`)

- **cli/** - Command-line interface (`tambo`)
  - Project scaffolding, component generation, and development utilities
  - Component registry auto-syncs to `/showcase/src/components/tambo/` from `/cli/src/registry/`
  - Built as ESM module with executable binary

- **showcase/** - Demo application (`@tambo-ai/showcase`)
  - Runs on port 8262
  - Next.js app demonstrating all Tambo components and patterns
  - Components auto-synced from CLI registry - edit CLI registry, not showcase components directly
  - Serves as both documentation and testing ground

- **docs/** - Documentation site (`@tambo-ai/docs`)
  - Runs on port 8263
  - Built with Fumadocs, includes comprehensive guides and API reference
  - This package contains ui components that originated from the cli/ package.
    Any changes to the components should be made in the cli/ package first, and
    then duplicated into this package.
  - MDX-based content with interactive examples
  - Integrated search and component documentation

- **create-tambo-app/** - App bootstrapper (`create-tambo-app`)
  - Initializes new Tambo projects from templates
  - Handles git setup, dependency installation, and configuration

- **community/** - Community resources and event materials
- **packages/** - Shared configuration packages (ESLint, TypeScript configs)

### Tambo Cloud Platform

- **apps/web** - Next.js app (UI) - runs on port 8260
- **apps/api** - NestJS app (OpenAPI server) - runs on port 8261
- **packages/db** - Drizzle ORM schema + migrations + DB helpers
- **packages/core** - Shared pure utilities (no DB access)
- **packages/backend** - LLM/agent-side helpers
- **packages/eslint-config, packages/typescript-config** - Shared tooling configs

### Prerequisites

- Node.js >=22
- npm >=11

It is OK to use `crypto.randomUUID()` without a fallback in runtime and test code (Node.js >=22 and modern browsers).

**Recommended:** Install [mise](https://mise.jdx.dev) for automatic version management. See [mise getting started](https://mise.jdx.dev/getting-started.html) for installation instructions.

### Tool Versions

Tool versions are managed via mise. Source of truth files:

- **Most tools**: `mise.toml`
- **Node.js**: `.node-version` (`.nvmrc` kept in sync for nvm compatibility)

These files are kept up to date by Renovate.

```bash
mise install              # Install/update tools to correct versions
mise exec -- <command>    # Preferred for scripts/CI/non-interactive shells
eval "$(mise activate)"   # Interactive shells only
```

**Changing tool versions**: Open a PR updating the authoritative version file(s). For Node.js, always update both `.node-version` and `.nvmrc` together. Run `mise install`, then verify with `npm run lint && npm run check-types && npm test`.

**Local overrides**: Use `.mise.local.toml` (gitignored) for local-only changes. Only for additive or patch-level changes—don't override Node.js or use incompatible versions.

## 2. Core Development Principles

### Philosophy

- **Move fast while maintaining high standards** - prioritize clarity and maintainability over cleverness.
- **Read the relevant code first**; follow existing patterns and naming.
- **Keep solutions small and simple**; favor functions over classes; avoid unnecessary abstractions.
- **Simplify Relentlessly**: Remove complexity aggressively - the simplest design that works is usually best.
- **Prefer immutability**. Don't mutate inputs; return new values. Use const, toSorted, object/array spreads.
- **Handle errors up-front** with guard clauses and early returns.

### Separation of Concerns

- **Keep business logic separate from UI components**.
- Extract business logic, calculations, and data transformations into separate files (`utils/`, `services/`, `lib/`).
- UI components should orchestrate, not implement complex logic.
- Makes testing easier and code more reusable.

### Fail-Fast, No Fallbacks


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tambo-ai/tambo](https://github.com/tambo-ai/tambo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
