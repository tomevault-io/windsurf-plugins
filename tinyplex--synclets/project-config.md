---
trigger: always_on
description: Synclets is a storage-agnostic sync engine development kit. It enables data synchronization between various storage backends (SQLite, PGlite, TinyBase, Files, Memory) over different transport layers (WebSockets, BroadcastChannel, Memory).
---

# Synclets Copilot Instructions

## Project Overview

Synclets is a storage-agnostic sync engine development kit. It enables data synchronization between various storage backends (SQLite, PGlite, TinyBase, Files, Memory) over different transport layers (WebSockets, BroadcastChannel, Memory).

## Architecture

- **Core (`src/core/synclet.ts`)**: The `Synclet` class is the central orchestrator. It composes:
  - **Data Connector**: Handles reading/writing of actual data (Atoms).
  - **Meta Connector**: Handles reading/writing of metadata (Timestamps/HLC).
  - **Transport**: Handles sending/receiving messages.
- **Connectors**: Implementations for specific storage engines.
  - **Browser Storage and Transport**: `src/browser/` (LocalStorage, SessionStorage, BroadcastChannel)
  - **Database Utilities**: `src/database/` - Shared SQL implementation used by `sqlite3` and `pglite`
  - **File/Directory Connectors**: `src/fs/`
  - **Memory Connectors and Transport**: `src/memory/`
  - **PGlite Connectors**: `src/pglite/`
  - **SQLite3 Connectors**: `src/sqlite3/`
  - **TinyBase Connectors**: `src/tinybase/`
- **Transports**: Implementations for communication channels.
  - **WebSocket Transport and Broker**: `src/ws/`
  - **Durable Object Transport**: `src/durable-object/`
- **Types (`src/@types/`)**: Type definitions are explicitly separated into this directory, mirroring the source structure.

## Key Concepts

- **Address**: A hierarchical path to a node, represented as an array of strings (e.g., `['users', '123', 'name']`).
- **Atom**: The leaf value at an address (string, number, boolean, null).
- **Timestamp**: Hybrid Logical Clock (HLC) strings used for conflict resolution (Last-Write-Wins).
- **Merkle Tree**: The sync protocol uses hashes of sub-trees to efficiently detect differences between peers.

## Development Workflows

- **Task Runner**: The project uses `gulp` for all build and maintenance tasks.
- **Commands**:
  - `npm run compileAndTestUnit`: Compiles and runs unit tests (`gulp compileAndTestUnit`).
  - `npm run lint`: Runs ESLint and Prettier (`gulp lint`).
  - `npm run ts`: Runs TypeScript type checking (`gulp ts`).
  - `npm run preCommit`: Runs the full suite of checks (lint, spell, ts, test, build).
- **Build**: `gulpfile.mjs` dynamically builds modules defined in `src/tsconfig.json`.
- **Code Reuse**: Always check for existing helper functions before implementing new functionality:
  - Look in the current file first (e.g., `gulpfile.mjs` has `execute()` for running commands)
  - Check `src/common/` for utility functions
  - Review similar implementations in the same file for patterns
  - Use established project patterns rather than reinventing solutions

## Coding Conventions

- **Functional Factories**: Prefer factory functions (e.g., `createSynclet`, `createSqlite3Connector`) over class inheritance.
- **Type Definitions**: Look in `src/@types` for interfaces. Implementation files often import types from `@synclets/@types/...`.
- **Documentation Convention**: Type definitions in `src/@types/index.d.ts` use triple-slash comments (`///`) as documentation labels that connect to corresponding entries in `src/@types/docs.js`. For example:
  - `/// TypeName` - Documents the type itself
  - `/// TypeName.propertyName` - Documents a property or method
  - The label in `index.d.ts` must exactly match the key in the `TYPES` object in `docs.js`
  - **Indentation matters**: Properties and methods must be indented in `docs.js` to match the alignment in the generated `index.d.ts` files
  - When adding new types or properties, update both files to avoid build errors
- **Database Abstraction**: When implementing a new SQL-based connector, adapt `createDatabaseConnector` (`src/database/common.ts`) instead of writing from scratch.
- **Utilities**: Use shared utilities from `src/common` (internal) and `src/utils` (public).

## Important Files

- `src/core/synclet.ts`: The core synchronization logic and protocol implementation.
- `src/database/common.ts`: Shared logic for SQL-based connectors.
- `gulpfile.mjs`: The build, test, and linting orchestration script.
- `src/tsconfig.json`: Defines the module structure and path mappings used by the build system.

## Code Style & Patterns

### Naming Conventions

1. **Type Aliases & Interfaces**
   - PascalCase for types: `Synclet`, `Connector`, `Transport`, `Atom`
   - Suffix with purpose: `Listener`, `Callback`, `Options`, `Config`

2. **Functions**
   - camelCase for all functions
   - Factory functions: `create` prefix (e.g., `createSynclet`, `createPgliteConnectors`)
   - Utility functions: descriptive verbs

3. **Constants**
   - UPPER_SNAKE_CASE for string constants
   - Used for configuration keys and internal identifiers

4. **Variables**
   - camelCase for all variables
   - Descriptive names preferred over abbreviations

### Utility Function Patterns

Following TinyBase patterns, use utility wrappers for consistency and tree-shaking:

1. **Array Operations**:

   ```typescript
   arrayForEach(array, callback); // instead of array.forEach

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tinyplex/synclets](https://github.com/tinyplex/synclets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
