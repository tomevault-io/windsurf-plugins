---
trigger: always_on
description: This document contains essential information for AI agents working on the libragen codebase.
---

# Agent Guidelines for libragen

This document contains essential information for AI agents working on the libragen codebase.

## Repository Structure

```
libragen/
├── packages/
│   ├── core/           # Core library - embeddings, search, vector store, library manager
│   ├── cli/            # Command-line interface
│   ├── mcp/            # Model Context Protocol server for AI assistants
│   ├── website/        # Documentation/marketing site (Astro + Starlight)
│   ├── collections-site/ # Collections browser site
│   └── ui/             # Shared UI components
├── schemas/            # JSON schemas for library metadata
└── collections/        # Official library collections
```

## Package Dependencies

```
@libragen/core  ← @libragen/cli
                ← @libragen/mcp
```

- `@libragen/core` is the foundation - changes here affect CLI and MCP
- `@libragen/cli` and `@libragen/mcp` are consumers of core

## Key Architectural Concepts

### Library Manager (`packages/core/src/manager.ts`)

The `LibraryManager` class handles library discovery, installation, and uninstallation.

**Path Discovery (IMPORTANT):**
- Default behavior: auto-detect `.libragen/libraries` in cwd + global directory
- Install: Defaults to Global directory (unless `-p` is used)
- Discovery: Project-local libraries take priority over global (first path wins)
- When `paths` option is provided (via `-p`): use ONLY those paths (no global, no auto-detection, no transformations applied by core, but CLI applies transformation)

```typescript
// Default: project (priority) + global
// Install defaults to global via getPrimaryDirectory() logic
const manager = new LibraryManager();

// Explicit paths only (no global, no auto-detection)
const manager = new LibraryManager({ paths: ['.libragen/libraries'] });
```
```

### CLI Commands (`packages/cli/src/commands/`)

All commands that work with installed libraries support the `-p, --path` flag:
- `list`, `install`, `uninstall`, `update`
- When `-p` is provided, ONLY those paths are used
- Multiple `-p` flags are additive: `-p path1 -p path2`

**The `--local` flag was REMOVED** - use `-p .libragen/libraries` instead.

### MCP Server (`packages/mcp/src/`)

The MCP server provides 8 tools:
1. `libragen_search` - Search libraries for relevant content
2. `libragen_list` - List available libraries
3. `libragen_build` - Build a library from source (async with worker threads)
4. `libragen_install` - Install a library or collection
5. `libragen_uninstall` - Remove an installed library
6. `libragen_update` - Update installed libraries
7. `libragen_collection` - Create collection files
8. `libragen_config` - Get configuration info (paths, version, discovered directories)

The MCP server discovers libraries from:
1. Workspace roots (via MCP roots capability) - checks for `.libragen/libraries`
2. Global directory (fallback)

**Key files:**
- `server.ts` - exports `getLibraryPaths()`, `updateLibraryPathsFromRoots()`
- `index.ts` - calls `updateLibraryPathsFromRoots()` after connection
- `tools/*.ts` - individual tool implementations
- `tasks/` - async build task management (TaskManager, WorkerPool, build-worker)

### Async Build System (`packages/mcp/src/tasks/`)

The `libragen_build` tool uses an async pattern to avoid MCP timeouts:

**Architecture:**
- `task-manager.ts` - Manages build tasks with queuing and concurrency control
- `worker-pool.ts` - Spawns and manages worker threads
- `build-worker.ts` - Runs build operations in worker threads

**How it works:**
1. `action: 'start'` creates a task and returns a `taskId` immediately
2. Build runs in a worker thread (up to n-1 CPU cores concurrently)
3. `action: 'status'` returns progress, current step, and result when complete
4. `action: 'cancel'` cancels a running or queued build

**Environment variables:**
- `LIBRAGEN_TASK_EXPIRY_MS` - How long completed tasks are retained (default: 1 hour)

## Documentation Locations

When making changes that affect user-facing behavior, update ALL of these:

1. **Package READMEs:**
   - `packages/cli/README.md`
   - `packages/core/README.md`
   - `packages/mcp/README.md`

2. **Website docs (`packages/website/src/content/docs/`):**
   - `getting-started.md` - Quick start, library storage
   - `cli.md` - CLI reference
   - `mcp.md` - MCP integration
   - `building.md` - Building libraries
   - `collections.md` - Collections

3. **Schemas (`schemas/`):**
   - `library-metadata.schema.json` - Library file format

## Testing

```bash
# Run all tests
npm test

# Run tests for specific package
npm test --workspace=@libragen/core
npm test --workspace=@libragen/cli
npm test --workspace=@libragen/mcp
```

Tests are in `__tests__/` directories within each package.

## Building

```bash
# Build everything (TypeScript + websites)
npm run build

# TypeScript only
npx tsc --build
```

## Releasing

This repo uses **changesets** with automated GitHub Actions for releases.

**DO NOT run `npm run version` or `npm run release` locally.** The release process is:

1. **Create a changeset** describing your changes:
   ```bash
   npm run changeset
   ```
   Or create a file manually in `.changeset/` with the format:
   ```markdown
   ---
   "@libragen/cli": minor
   ---

   Description of changes
   ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [libragen/libragen](https://github.com/libragen/libragen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
