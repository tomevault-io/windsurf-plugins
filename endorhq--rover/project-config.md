---
trigger: always_on
description: This file provides guidance to AI agents (Claude Code, Cursor, etc.) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents (Claude Code, Cursor, etc.) when working with code in this repository.

## Project Overview

Rover is a TypeScript-based workspace that helps developers and AI agents spin up services instantly. The project is organized as a **monorepo** with multiple packages, each serving different purposes in the Rover ecosystem.

### Repository Structure

```
rover/
├── packages/
│   ├── cli/          # Main CLI tool (TypeScript)
│   ├── extension/    # VS Code extension (TypeScript)
│   └── telemetry/    # Telemetry library (TypeScript)
├── package.json      # Root workspace configuration
└── AGENTS.md        # This file
```

## Essential Development Commands

### Workspace Commands (run from root)

```bash
# Development workflow
pnpm dev              # Start all packages in development mode

# Building
pnpm build            # Build all packages

# Testing
pnpm test             # Run tests for all packages

# Type checking
pnpm check            # TypeScript type checking for all packages

# Linting & Formatting
pnpm lint             # Lint all files (with auto-fix)
pnpm format           # Format all files
```

### Package-Specific Commands

```bash
# CLI package (packages/cli/)
cd packages/cli
pnpm dev           # Development mode with watch
pnpm build         # Type-check and build
pnpm check         # TypeScript type checking
pnpm test          # Run tests with Vitest
pnpm test:watch    # Run tests in watch mode
pnpm test:ui       # Open Vitest UI
pnpm test:coverage # Run tests with coverage

# Extension package (packages/extension/)
cd packages/extension
pnpm compile       # Compile TypeScript
pnpm watch         # Watch mode for development
pnpm package       # Build for production
pnpm test          # Run VS Code extension tests

# Telemetry package (packages/telemetry/)
cd packages/telemetry
pnpm dev           # Development mode with watch
pnpm build         # Build for production
pnpm check         # TypeScript type checking
```

## Architecture

### CLI Package (`packages/cli/`)

- **Entry point**: `src/index.ts` - Sets up the CLI using Commander.js
- **Commands**: `src/commands/` - Each command is implemented as a separate module
- **Build output**: `dist/index.mjs` - Single bundled ES module file
- **Libraries**: `src/lib/` - Core functionality (Git, Docker, AI agents, configs)
- **Utilities**: `src/utils/` - Helper functions and utilities
- **Testing**: Uses Vitest with real Git operations and mocked external dependencies

Key architectural decisions:

- Uses tsdown for bundling
- AI providers implement a common interface for easy switching between Claude and Gemini
- Commands interact with Git worktrees for isolated task execution
- Docker containers execute AI agent tasks

### Extension Package (`packages/extension/`)

- **Entry point**: `src/extension.mts` - VS Code extension activation
- **Providers**: Tree data providers for Rover tasks
- **Panels**: Webview panels for detailed task information
- **Views**: Lit-based webview components
- **CLI Integration**: Communicates with Rover CLI via child processes

### Telemetry Package (`packages/telemetry/`)

- **Shared telemetry library** used by CLI and extension
- **Event tracking** for usage analytics
- **Privacy-focused** implementation

## Technical Details

- **TypeScript**: Strict mode enabled, targeting ES2022
- **Module system**: ES modules with Node.js compatibility
- **Node version**: Requires Node.js 20+ and pnpm 10+ (see root package.json engines)
- **Monorepo**: Uses pnpm workspaces for package management

### Process Execution

**Always use `launch` (async) and `launchSync` from `packages/core/src/os.ts` to spawn processes. Do not use the Node.js `child_process` API directly.**

These functions are wrappers around [execa](https://github.com/sindresorhus/execa) that provide consistent behavior across the codebase:

- `launch(command, args?, options?)` - Async process execution. Returns an execa result object.
- `launchSync(command, args?, options?)` - Synchronous process execution. Returns an execa sync result object.

Key behaviors handled by these wrappers:
- Proper argument escaping via `parseCommandString` and execa template strings
- Detached process groups by default (prevents child termination on parent signals)
- Verbose logging when `VERBOSE` is enabled
- Consistent stdio option expansion

## Build Pipeline

The project uses tsdown for bundling with two distinct build modes controlled by the `TSUP_DEV` environment variable.

### Production Build (`pnpm build`)

- **Minified output** for smaller bundle size
- **No source maps** generated
- **Workspace packages kept external** (imported from their `dist/` folders)
- Used for releases and distribution

### Development Build (`pnpm build-dev` or `pnpm dev`)

- **No minification** for readable stack traces
- **Source maps enabled** pointing to original TypeScript files
- **Workspace packages bundled and aliased** to their TypeScript source (`src/index.ts`)
- Enables full debugging with accurate line numbers across all packages

Key differences in `packages/cli/tsdown.config.ts`:

| Feature     | Production | Development                    |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [endorhq/rover](https://github.com/endorhq/rover) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
