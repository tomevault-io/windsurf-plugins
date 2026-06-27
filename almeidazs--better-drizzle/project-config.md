---
trigger: always_on
description: > Meta note: This is the primary agent knowledge base file for this repository. When learning something about the codebase that will help with future tasks, update this file directly.
---

# Better-Drizzle Repository – Agent Field Notes

> Meta note: This is the primary agent knowledge base file for this repository. When learning something about the codebase that will help with future tasks, update this file directly.

- **Repository scope**: `better-drizzle` is a small Bun/TypeScript workspace focused on a single core package, `packages/core`, plus a benchmark suite used to measure API-parity performance and memory overhead against raw Drizzle ORM.
- **Workspace layout**:
  - `packages/core`: the published library
  - `packages/soft-delete`: official soft delete plugin
  - `packages/timestamps`: official timestamps plugin
  - `benchmark`: Bun + SQLite benchmark suite
  - `examples`: Markdown-only example catalog and usage guides
  - `README.md`: project-level documentation
  - `packages/core/README.md`: package-level documentation, currently intentionally kept in sync with the root README
- **Package manager and runtime**: Bun is the primary runtime for local commands and benchmarks. The workspace is configured as a TypeScript ESM monorepo.
- **Package publishing/build**:
  - all published workspace libraries now build to `dist/`
  - each package emits `dist/index.js` (ESM), `dist/index.cjs` (CommonJS), and `dist/index.d.ts`
  - root build entrypoint is `scripts/build.ts`, powered by `Bun.build` plus `tsc` declaration emit
  - package manifests publish only `dist`, `README.md`, and `LICENSE`
- **Top-level scripts**:
  - `bun run bench`: run the time benchmark suite
  - `bun run bench:memory`: run the memory/overhead benchmark suite
  - `bun run bench:all`: run both benchmark suites
- **Core dependencies**:
  - `drizzle-orm` as a peer dependency
  - `typescript` as a peer dependency
  - `mitata` for benchmarking
  - `@biomejs/biome` for formatting and linting

## Architecture

- **Entry point**: `packages/core/src/index.ts`
  - Exports `better(...)`
  - Exports `definePlugin(...)`
  - Delegates root/transaction client binding to `packages/core/src/shared/client/factory.ts`
  - Builds a base runtime context once
  - Initializes plugins once during bootstrap
  - Re-binds delegates/extensions per bound client (`db` or `tx`) without re-running plugin setup
  - Registers repositories by TypeScript table key and database table name
- **Runtime layout**:
  - `packages/core/src/shared/client/context.ts`: builds the runtime context and precomputed table metadata
  - `packages/core/src/shared/client/delegate.ts`: exposes the delegate methods for each table
  - `packages/core/src/shared/client/factory.ts`: binds root and transaction clients, retries, nested savepoints, and transaction lifecycle hooks
  - `packages/core/src/shared/client/operations.ts`: main query and mutation execution paths; this is the hottest file for performance work
  - `packages/core/src/shared/client/hooks.ts`: optional hook execution
  - `packages/core/src/shared/client/plugins.ts`: plugin initialization, validation, transform pipeline, and extension application
  - `packages/core/src/shared/query/compiler.ts`: compiles typed `where`, `select`, `include`, `orderBy`, and pagination inputs into Drizzle-compatible query pieces
  - `packages/core/src/shared/errors.ts`: shared error helpers
  - `packages/core/src/types/*`: public type surface
- **No internal runtime package**: the old `packages/core/src/internal/runtime.ts` was removed. Runtime logic now lives under `shared/client` and `shared/query`.

## Design intent

- **Primary goal**: give Drizzle users a minimal repository-style API without hiding Drizzle or rebuilding a full ORM on top of it.
- **Non-goals**:
  - not replacing raw Drizzle for fully manual query work
  - not adding broad abstraction layers
  - not adding runtime magic that duplicates schema knowledge
- **Bias**: prefer simpler code, fewer branches, fewer allocations, fewer helpers, and fewer layers.

## API surface

- **Table delegates expose**:
  - `findMany`
  - `findFirst`
  - `findOne`
  - `findUnique`
  - `create`
  - `createMany`
  - `update`
  - `updateMany`
  - `delete`
  - `deleteMany`
  - `upsert`
  - `upsertMany`
  - `count`
  - `exists`
  - `paginate`
  - `$withState`
  - `$withoutPlugins`
- **Create conflict handling**:
  - `create` and `createMany` accept `skipDuplicates`
  - supported forms: `true` or `readonly ColumnName[]`
  - `skipDuplicates: true` makes `create` return `null` when the insert is skipped
  - `createMany.count` reflects only rows actually inserted when conflicts are ignored
  - explicit column arrays map to schema column names; targeted duplicate-skip is intentionally dialect-sensitive
- **Client-level lookup**:
  - `repository(name)` resolves by schema key or db table name
- **Transactions**:
  - `db.transaction(callback, options?)` is the official API
  - transaction clients are full Better Drizzle clients with `transaction`, `rollback`, `afterCommit`, and `afterRollback`
  - transaction context lives on the runtime context; operation/plugin hooks can read `isInTransaction`, `transaction`, and `transactionContext`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [almeidazs/better-drizzle](https://github.com/almeidazs/better-drizzle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
