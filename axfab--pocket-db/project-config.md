---
trigger: always_on
description: This file provides guidance to Claude when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude when working with code in this repository.

## Project Synopsis

Pocket DB is an embedded NoSQL document database for Node.js, persisted in a single append-only file. The design is inspired by SQLite (single-file, embedded) and MongoDB (document model, familiar API), but intentionally compact. Target use cases: desktop apps, CLI tools, Electron apps, prototypes, local servers, plugins, structured caches.

The core constraint is **never reserializing the entire database on write**. All writes are append-only records; the in-memory state is rebuilt by replaying the log at open time.

## Commands

```bash
npm install          # install dependencies (typescript, @types/node, eslint, tsx)
npm run build        # compile TypeScript → dist/ (ESM + CJS, see below)
npm test             # run all tests directly from .ts via tsx, no build needed
npm run test:coverage # same, with Node's native coverage instrumentation
npm run bench        # run the benchmark suite (separate "benchmarks" workspace)
npm run lint         # eslint .
```

To run a single test file:
```bash
node --import tsx --test "tests/indexes.test.ts"
```

To run tests matching a name pattern:
```bash
node --import tsx --test --test-name-pattern="creates a string index" "tests/*.test.ts"
```

Benchmarks live in `benchmarks/`, a separate npm workspace (`pocket-db-benchmarks`) so
comparison-only dependencies (better-sqlite3, lokijs, lowdb) never pollute the
published package's own `devDependencies`.

## Architecture Overview

### Layer Diagram

```
src/api/          ← public surface: open(), Database, Collection, Cursor, DocumentCache
src/search/       ← query compilation, evaluation, sort, document update operators
src/indexes/      ← primary index, secondary indexes (string/number), IndexManager
src/storage/      ← file format, binary encoding, CRC32 validation, file lock
src/native/       ← reserved for future C backend (empty, interfaces only)
```

### File Storage (`src/storage/`)

`FileStorage` is the lowest layer. It wraps a synchronous Node.js file descriptor (`node:fs` sync API — `readSync`, `writeSync`). Every write goes through `appendOperation(identifier, payload)`, which:
1. uses the in-memory `currentOffset` counter to get the write offset (initialized from `fstatSync` once at open, then maintained internally),
2. encodes the record as `[4-byte identifier][4-byte payload length][N-byte payload][4-byte CRC32]`,
3. writes it atomically (retry loop), advances `currentOffset` by `record.byteLength`, and returns the offset.

Reading a document later means calling `readOperationAtOffset(offset)` — only the bytes for that one record are read from disk.

**Bulk-read optimization (`readBulkRange(offsets)`):** given a set of offsets, reads the minimum contiguous span covering all of them — from the lowest offset to the end of the record at the highest (one extra small header read determines that record's exact length) — never the whole file. Used by the cursor's multi-candidate path (query planner selects ≥ 2 candidates) *when it's worth doing so lazily/adaptively* — see the Cursor section's "Lazy/adaptive bulk-range read" below and [ADR 0018](docs/adr/0018-lazy-bulk-read-by-limit.md) — and unconditionally by `rebuildIndex()`/`refreshIndexesAfterCompaction()` to backfill/refresh a collection's secondary indexes from every existing document in one read instead of one `readOperationAtOffset` call per document. Single-candidate reads still use `readOperationAtOffset` directly. See [ADR 0016](docs/adr/0016-bounded-candidate-range-read.md).

**Streaming replay (`readOperations()`):** a generator over a bounded sliding read window (default 8MiB, `DEFAULT_REPLAY_CHUNK_BYTES`), not a whole-file `Buffer` — peak memory during `open()`/`stats()`/`compact()` is a small multiple of the window size, not the file size. See [ADR 0017](docs/adr/0017-streaming-replay-buffer.md).

**Binary encoding conventions:**
- All multi-byte integers are big-endian.
- Payloads are zero-padded to 4-byte alignment.
- `U29` (in `u29.ts`) is a variable-length 1–4 byte unsigned 29-bit integer used for string/JSON byte lengths, using the same high-bit continuation scheme as UTF-8.
- CRC32 is computed over `identifier + length_field + payload` (everything before the checksum).

**Operation identifiers** (4 ASCII bytes, defined in `constants.ts`):
| Magic | Meaning |
|-------|---------|
| `ncl1` | New collection |
| `dco1` | Drop collection |
| `idx1` | Create index |
| `dix1` | Drop index |
| `put1` | Put document (insert/replace/update — always a full document) |
| `del1` | Delete document |
| `txnb` | Transaction begin |
| `txnc` | Transaction commit |
| `hol0` | Hole marker (used by compaction to pad dead space; silently skipped on replay) |

### File Lock (`src/storage/file-lock.ts`)

`FileLock.acquire(dbPath)` creates a `.lock` file next to the database using `openSync` with the `wx` (exclusive create) flag. The lock file contains the process PID. On conflict it reads the existing PID and checks if that process is alive via `process.kill(pid, 0)`. A stale lock (dead process) is removed and the acquire retried. `lock.release()` unlinks the lock file.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AxFab/pocket-db](https://github.com/AxFab/pocket-db) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
