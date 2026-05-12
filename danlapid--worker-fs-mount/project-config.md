---
trigger: always_on
description: This file provides guidance to Claude Code when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

**worker-fs-mount** is an npm package that allows Cloudflare Workers to mount WorkerEntrypoints as virtual filesystems. It provides a drop-in replacement for `node:fs/promises` that intercepts filesystem calls and redirects mounted paths to WorkerEntrypoint implementations via jsrpc.

### Key Concept

Users call `mount('/mnt/path', stub)` where `stub` is a WorkerEntrypoint (from `ctx.exports`, `env.SERVICE`, or a Durable Object stub). After mounting, any `node:fs/promises` operation targeting that path is forwarded to the stub's methods.

```typescript
import { env } from 'cloudflare:workers';
import { mount } from 'worker-fs-mount';
import fs from 'node:fs/promises';  // Aliased to our implementation

// Mount at module level using importable env
mount('/mnt/storage', env.STORAGE_SERVICE);

export default {
  async fetch(request) {
    await fs.readFile('/mnt/storage/file.txt');  // → calls env.STORAGE_SERVICE.readFile('/file.txt')
  }
};
```

**Global/module-level mounts** (preferred): Mount at module scope using `import { env, exports } from 'cloudflare:workers'`. Works for R2, KV, service bindings, and same-worker entrypoints. Simple, no cleanup needed.

**Request-scoped mounts**: Required for Durable Objects (getting a DO stub is IO and requires request scope). Use `withMounts` when different requests need different mounts (e.g., per-user DOs) to prevent mount collisions.

## Project Structure

This is a pnpm monorepo with the following structure:

```
worker-fs-mount/
├── packages/
│   ├── worker-fs-mount/         # Main package - mount system and fs/promises replacement
│   │   ├── src/
│   │   │   ├── index.ts         # Entry point - exports public API (mount, withMounts, etc.)
│   │   │   ├── fs-promises.ts   # Drop-in replacement for node:fs/promises
│   │   │   ├── types.ts         # TypeScript interfaces (WorkerFilesystem, Stat, DirEntry)
│   │   │   ├── utils.ts         # Shared utilities (createFsError, normalizePath, etc.)
│   │   │   └── registry.ts      # Mount registry - mount(), unmount(), withMounts(), isMounted()
│   │   └── README.md
│   ├── durable-object-fs/       # Durable Object filesystem implementation (SQLite storage)
│   ├── r2-fs/                   # R2 bucket filesystem implementation
│   ├── memory-fs/               # In-memory filesystem implementation
│   └── tests/                   # Integration tests
│       ├── index.ts             # Test worker entry point
│       ├── index.test.ts        # Integration tests using vitest
│       └── wrangler.toml        # Test worker config with alias
├── examples/
│   ├── durable-object-backed-fs/  # Example using durable-object-fs
│   └── r2-backed-fs/              # Example using r2-fs
├── package.json                 # Root workspace scripts
├── pnpm-workspace.yaml          # pnpm workspace configuration
└── CLAUDE.md                    # This file
```

## Build & Development Commands

From the root directory:

```bash
pnpm install         # Install dependencies (uses pnpm workspaces)
pnpm build           # Compile TypeScript to dist/
pnpm dev             # Watch mode compilation
pnpm typecheck       # Type check without emitting
pnpm test            # Run integration tests
pnpm clean           # Remove dist/
```

## Architecture

### How Module Aliasing Works

The package uses wrangler's `[alias]` feature to replace `node:fs/promises` at build time:

1. Users add an alias in their `wrangler.toml`:
   ```toml
   [alias]
   "node:fs/promises" = "worker-fs-mount/fs"
   ```

2. **`fs-promises.ts`** exports all standard `node:fs/promises` functions
3. Each function checks if the path is under a mount:
   - If mounted: calls the stub's method via jsrpc and returns the result
   - If not mounted: calls the real `node:fs/promises` method (via `node:fs` sync module's `.promises`)

### Mount Registry (`registry.ts`)

- `mounts`: `Map<string, Mount>` - stores active mounts keyed by normalized path
- `mount(path, stub)`: validates path, checks for conflicts, adds to registry
- `findMount(path)`: iterates mounts to find longest matching prefix
- `normalizePath(path)`: removes trailing slashes, collapses multiple slashes

### Type System (`types.ts`)

- **`WorkerFilesystem`**: stream-first interface that mounted stubs must implement
  - Required (6): `stat`, `createReadStream`, `createWriteStream`, `readdir`, `mkdir`, `rm`
  - Optional (2): `symlink`, `readlink`
  - Derived operations (implemented in fs-promises.ts): `readFile`, `writeFile`, `truncate`, `rename`, `cp`, `unlink`, `access`, `appendFile`
- **`Stat`**: `{ type: 'file'|'directory'|'symlink', size: number, lastModified?: Date, ... }`
- **`DirEntry`**: `{ name: string, type: 'file'|'directory'|'symlink' }`

## Key Implementation Details

### Path Handling

- All mount paths must be absolute (start with `/`)
- Paths are normalized: trailing slashes removed, multiple slashes collapsed
- Reserved paths (`/bundle`, `/tmp`, `/dev`) cannot be mounted over
- Nested mounts are not allowed (can't mount `/a/b` if `/a` is mounted)

### Type Conversions (in fs-promises.ts)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danlapid/worker-fs-mount](https://github.com/danlapid/worker-fs-mount) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
