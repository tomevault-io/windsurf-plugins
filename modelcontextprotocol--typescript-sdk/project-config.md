---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

```sh
pnpm install         # Install all workspace dependencies

pnpm build:all       # Build all packages
pnpm lint:all        # Run ESLint + Prettier checks across all packages
pnpm lint:fix:all    # Auto-fix lint and formatting issues across all packages
pnpm typecheck:all   # Type-check all packages
pnpm test:all        # Run all tests (vitest) across all packages
pnpm check:all       # typecheck + lint across all packages

# Run a single package script (examples)
# Run a single package script from the repo root with pnpm filter
pnpm --filter @modelcontextprotocol/core test                # vitest run (core)
pnpm --filter @modelcontextprotocol/core test:watch          # vitest (watch)
pnpm --filter @modelcontextprotocol/core test -- path/to/file.test.ts
pnpm --filter @modelcontextprotocol/core test -- -t "test name"
```

## Breaking Changes

When making breaking changes, document them in **both**:

- `docs/migration.md` — human-readable guide with before/after code examples
- `docs/migration-SKILL.md` — LLM-optimized mapping tables for mechanical migration

Include what changed, why, and how to migrate. Search for related sections and group related changes together rather than adding new standalone sections.

## Code Style Guidelines

- **TypeScript**: Strict type checking, ES modules, explicit return types
- **Naming**: PascalCase for classes/types, camelCase for functions/variables
- **Files**: Lowercase with hyphens, test files with `.test.ts` suffix
- **Imports**: ES module style, include `.js` extension, group imports logically
- **Formatting**: 2-space indentation, semicolons required, single quotes preferred
- **Testing**: Place tests under each package's `test/` directory (vitest only includes `test/**/*.test.ts`), use descriptive test names
- **Comments**: JSDoc for public APIs, inline comments for complex logic

### JSDoc `@example` Code Snippets

JSDoc `@example` tags should pull type-checked code from companion `.examples.ts` files (e.g., `client.ts` → `client.examples.ts`). Use `` ```ts source="./file.examples.ts#regionName" `` fences referencing `//#region regionName` blocks; region names follow `exportedName_variant` or `ClassName_methodName_variant` pattern (e.g., `applyMiddlewares_basicUsage`, `Client_connect_basicUsage`). For whole-file inclusion (any file type), omit the `#regionName`.

Run `pnpm sync:snippets` to sync example content into JSDoc comments and markdown files.

## Architecture Overview

### Core Layers

The SDK is organized into three main layers:

1. **Types Layer** (`packages/core/src/types/types.ts`) - Protocol types generated from the MCP specification. All JSON-RPC message types, schemas, and protocol constants are defined here using Zod v4.

2. **Protocol Layer** (`packages/core/src/shared/protocol.ts`) - The abstract `Protocol` class that handles JSON-RPC message routing, request/response correlation, capability negotiation, and transport management. Both `Client` and `Server` extend this class.

3. **High-Level APIs**:
    - `Client` (`packages/client/src/client/client.ts`) - Client implementation extending Protocol with typed methods for MCP operations
    - `Server` (`packages/server/src/server/server.ts`) - Server implementation extending Protocol with request handler registration
    - `McpServer` (`packages/server/src/server/mcp.ts`) - High-level server API with simplified resource/tool/prompt registration

### Public API Exports

The SDK has a two-layer export structure to separate internal code from the public API:

- **`@modelcontextprotocol/core`** (main entry, `packages/core/src/index.ts`) — Internal barrel. Exports everything (including Zod schemas, Protocol class, stdio utils). Only consumed by sibling packages within the monorepo (`private: true`).
- **`@modelcontextprotocol/core/public`** (`packages/core/src/exports/public/index.ts`) — Curated public API. Exports only TypeScript types, error classes, constants, and guards. Re-exported by client and server packages.
- **`@modelcontextprotocol/client`** and **`@modelcontextprotocol/server`** (`packages/*/src/index.ts`) — Final public surface. Package-specific exports (named explicitly) plus re-exports from `core/public`.

When modifying exports:
- Use explicit named exports, not `export *`, in package `index.ts` files and `core/public`.
- Adding a symbol to a package `index.ts` makes it public API — do so intentionally.
- Internal helpers should stay in the core internal barrel and not be added to `core/public` or package index files.
- The package root entry must stay runtime-neutral so browser and Cloudflare Workers bundlers can consume it. Exports whose module graph transitively touches unpolyfillable Node builtins (`node:child_process`, `node:net`, `cross-spawn`, etc.) must live at a named subpath export (e.g. `./stdio`) and be covered by a `barrelClean` test in that package.

### Transport System

Transports (`packages/core/src/shared/transport.ts`) provide the communication layer:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [modelcontextprotocol/typescript-sdk](https://github.com/modelcontextprotocol/typescript-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
