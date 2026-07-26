---
trigger: always_on
description: **node-json-db** is a lightweight, file-based JSON database for Node.js/TypeScript with zero production dependencies. Key features:
---

# Repository Instructions for GitHub Copilot

## Project Overview

**node-json-db** is a lightweight, file-based JSON database for Node.js/TypeScript with zero production dependencies. Key features:

- **DataPath navigation**: XPath-like syntax (`/users/0/name`, `/items[]`, `/data[-1]`) to navigate nested JSON data
- **Async-first API**: All database operations return Promises (async/await)
- **Optional AES-256-GCM encryption**: Via `config.setEncryption(key32bytes)`, stored as `.enc.json`
- **Concurrency safety**: Built-in reader-writer locks prevent race conditions
- **Pluggable adapters**: `IAdapter<T>` interface for custom storage backends
- **Automatic persistence**: `saveOnPush: true` (default) writes to disk on every `push()`/`delete()`
- **Generic type-safe retrieval**: `getObject<T>(path)` and `getObjectDefault<T>(path, default)`

---

## Package Manager

This project uses **pnpm** as the package manager, NOT npm.

### Setup Requirements

1. **Node.js**: Use the latest LTS version of Node.js
2. **pnpm**: Use Corepack to enable pnpm (preferred method)

### Installing Dependencies

```bash
# Enable corepack (comes with Node.js 16.13+)
corepack enable

# Install dependencies
pnpm install
```

If corepack is not available:
```bash
npm install -g pnpm
pnpm install
```

### Important Rules

- **NEVER commit `package-lock.json`** - This file should not exist in this repository
- Always use `pnpm` commands, not `npm`:
  - `pnpm install` instead of `npm install`
  - `pnpm run build` instead of `npm run build`
  - `pnpm test` instead of `npm test`
- The project specifies the exact pnpm version in `package.json` under `packageManager`
- Only commit `pnpm-lock.yaml` for dependency locks

### Common Commands

```bash
pnpm run build     # Build the project (TypeScript → dist/)
pnpm test          # Run all tests with coverage (jest --coverage)
pnpm run build:doc # Generate TypeDoc API documentation (→ docs/)
```

---

## Architecture & Directory Structure

```
node-json-db/
├── src/                              # TypeScript source (compiled to dist/)
│   ├── JsonDB.ts                    # Main exported class — all public API methods
│   ├── adapter/
│   │   ├── IAdapter.ts              # Interface: readAsync() / writeAsync()
│   │   ├── data/
│   │   │   └── JsonAdapter.ts       # JSON serialize/deserialize + date revival
│   │   └── file/
│   │       ├── FileAdapter.ts       # Raw filesystem I/O
│   │       └── CipheredFileAdapter.ts  # AES-256-GCM encryption layer
│   ├── lib/
│   │   ├── JsonDBConfig.ts          # Config & ConfigWithAdapter classes
│   │   ├── Errors.ts                # DatabaseError / DataError (extend NestedError)
│   │   ├── ArrayInfo.ts             # Array path parsing (e.g. items[0], items[])
│   │   ├── DBParentData.ts          # Parent node resolution for writes/deletes
│   │   └── Utils.ts                 # merge(), removeTrailingChar(), KeyValue type
│   └── lock/
│       ├── Lock.ts                  # readLockAsync() / writeLockAsync() helpers
│       ├── ReadWriteLock.ts         # High-perf reader-writer lock with pooling
│       └── Error.ts                 # TimeoutError class
├── test/                            # Jest test suite (ts-jest)
│   ├── 01-utils.test.ts            # ArrayInfo regex + safe-regex ReDoS tests
│   ├── 02-jsondb.test.ts           # Core CRUD, merge, types, errors
│   ├── 03-existing-db.test.ts      # File persistence and reload
│   ├── 04-array-utils.test.ts      # Array indexing, append, nested arrays
│   ├── 05-errors-test.ts           # Error types and DataPath error codes
│   ├── 06-concurrency.test.ts      # Lock timeouts, concurrent access
│   ├── 07-cyphered.test.ts         # Encryption key validation, encrypt/decrypt
│   ├── ArrayInfo.test.ts           # Unit tests for ArrayInfo class
│   ├── DBParentData.test.ts        # Unit tests for DBParentData class
│   ├── JsonDB.test.ts              # Unit tests for JsonDB class
│   ├── adapter/
│   │   └── adapters.test.ts        # Unit tests for adapters
│   └── lock/                       # Lock unit tests
├── dist/                            # Compiled JS output (git-ignored, built by tsc)
├── jest.config.js                   # Jest configuration (preset: ts-jest)
├── tsconfig.json                    # TypeScript config (strict, commonjs, esnext)
└── package.json                     # Scripts, devDependencies, commitlint config
```

**Adapter chain**: `JsonDB → JsonAdapter → (CipheredFileAdapter | FileAdapter)`

---

## TypeScript Conventions

- **Strict mode** is enabled (`"strict": true` in `tsconfig.json`) — no implicit `any`
- **Target**: `esnext` compiled to `commonjs` for Node.js compatibility
- **Naming**:
  - Classes/Interfaces: `PascalCase` (e.g., `JsonDB`, `ArrayInfo`, `IAdapter`)
  - Private class members: prefixed with `_` (e.g., `_filename`)
  - Methods/variables: `camelCase`
- **Visibility**: Mark implementation details `private`, keep the public API minimal
- **Generics**: Use generic type parameters for type-safe retrieval (e.g., `getObject<T>()`)
- **Imports**: Named imports from relative paths; no barrel `index.ts` files
- **JSDoc**: Required for all public API methods and configuration options

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Belphemur/node-json-db](https://github.com/Belphemur/node-json-db) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
