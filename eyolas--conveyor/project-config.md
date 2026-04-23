---
trigger: always_on
description: Multi-backend TypeScript job queue supporting PostgreSQL, SQLite, and in-memory stores. Deno 2
---

# Conveyor

Multi-backend TypeScript job queue supporting PostgreSQL, SQLite, and in-memory stores. Deno 2
monorepo with 8 workspace packages, published on JSR (v1.0.0).

BullMQ-like API without requiring Redis. See `prd.md` for full specs.

## Guiding Principles

- **Zero lock-in**: switching backends = changing one line of config
- **Familiar API**: if you know BullMQ, you know Conveyor
- **Runtime agnostic**: Deno 2, Node.js 18+, and Bun first-class
- **Type-safe**: strict TypeScript, generics on payloads
- **Testable**: in-memory store makes tests fast and deterministic
- **No runtime-specific APIs in core**: only Web Standards APIs (`setTimeout`, `EventTarget`,
  `crypto.randomUUID`)

## Packages

| Package                       | Path                         | Description                         |
| ----------------------------- | ---------------------------- | ----------------------------------- |
| `@conveyor/core`              | `packages/core`              | Queue, Worker, FlowProducer, events |
| `@conveyor/shared`            | `packages/shared`            | Shared types, utils, StoreInterface |
| `@conveyor/store-memory`      | `packages/store-memory`      | In-memory store                     |
| `@conveyor/store-pg`          | `packages/store-pg`          | PostgreSQL store                    |
| `@conveyor/store-sqlite-core` | `packages/store-sqlite-core` | SQLite base (shared logic)          |
| `@conveyor/store-sqlite-node` | `packages/store-sqlite-node` | SQLite for Node                     |
| `@conveyor/store-sqlite-bun`  | `packages/store-sqlite-bun`  | SQLite for Bun                      |
| `@conveyor/store-sqlite-deno` | `packages/store-sqlite-deno` | SQLite for Deno                     |

## Commands

```bash
deno task test              # Run all tests (vitest)
deno task test:core         # Core + conformance tests
deno task test:memory       # Memory store conformance tests
deno task test:pg           # PostgreSQL store tests (needs docker)
deno task test:sqlite:node  # SQLite Node tests
deno task test:sqlite:bun   # SQLite Bun tests (uses bun test)
deno task test:sqlite:deno  # SQLite Deno tests
deno task bench             # Run benchmarks
deno task lint              # deno lint (recommended rules)
deno task fmt               # deno fmt
deno task check             # Type-check all package entry points
deno task setup             # Set up git hooks
```

PostgreSQL tests require a running database:

```bash
docker-compose up -d  # Start PG container
```

## Code Conventions

### Style & Formatting

- **Formatter:** `deno fmt` — lineWidth 100, indentWidth 2, singleQuote true
- **Linter:** `deno lint` — recommended rules
- **TypeScript:** strict mode + `noUncheckedIndexedAccess`

### Naming

| Element             | Convention                | Example                          |
| ------------------- | ------------------------- | -------------------------------- |
| Classes             | PascalCase                | `Queue`, `Worker`, `MemoryStore` |
| Functions/variables | camelCase                 | `parseDelay`, `createJobData`    |
| Constants           | UPPER_SNAKE_CASE          | `QUEUE_NAME_RE`                  |
| Types/Interfaces    | PascalCase, no `I` prefix | `JobData`, `StoreInterface`      |
| DB columns          | snake_case                | `queue_name`, `created_at`       |
| Files               | kebab-case                | `memory-store.ts`                |

### Imports & Exports

- Separate `import type` from runtime imports; types come first
- Barrel exports via `mod.ts`
- Separate `export type` from `export`

### File Organization

- One class per file
- Section separators: `// ─── Section Name ─────────────────────`
- Order: properties → constructor → public methods → private methods

### Patterns

- Generic defaults to `unknown`: `class Queue<T = unknown>`
- `interface` for contracts, `type` for unions/aliases
- `readonly` / `private readonly` for immutability
- `Symbol.asyncDispose` for cleanup
- `structuredClone()` for defensive copies
- Readable numbers: `30_000`

### JSDoc

- `@module` at top of every file
- Tags: `@typeParam`, `@param`, `@returns`, `@throws`, `@example`
- `{@linkcode Type}` for cross-references
- `/** @internal */` for internal-only types

### Errors

- `Error` with descriptive messages at boundaries
- `RangeError` for range validations
- Event handlers wrapped in try-catch + `onEventHandlerError` callback

### Tests

- Vitest (all runtimes except Bun which uses `bun test`)
- Files named `*.test.ts`
- Test names: `test('Class.method description', async () => ...)`
- Helpers at top of file (`createQueue()`, `createWorker()`)
- Section separators: `// ─── Feature ─────`
- Always cleanup: call `close()`, `disconnect()`

### Stores

- `implements StoreInterface` (no abstract class except SQLite base)
- Options extend `StoreOptions`
- PG: tagged template literals, `SELECT ... FOR UPDATE SKIP LOCKED` for locking, `LISTEN/NOTIFY` for
  events
- SQLite: prepared statements with named parameters, WAL mode + `BEGIN IMMEDIATE`, polling for
  events
- Memory: `Map` + mutex for locking, `EventEmitter` for events
- Core never depends on a concrete driver — each store encapsulates its runtime-specific driver


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eyolas) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
