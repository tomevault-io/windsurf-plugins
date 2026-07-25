---
trigger: always_on
description: This file provides guidance to AI coding agents (Claude Code, Codex, Copilot, Cursor, Aider, etc.) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Codex, Copilot, Cursor, Aider, etc.) when working with code in this repository.

## Project Overview

This is the official MongoDB Node.js driver (`mongodb` npm package). It provides a TypeScript/JavaScript interface for applications to interact with MongoDB deployments. The driver implements the cross-driver MongoDB specifications.

## Related Repositories

- **[mongodb/specifications](https://github.com/mongodb/specifications)** — Cross-driver MongoDB specifications. **This is the source of truth** for behavior the driver must implement (CRUD, SDAM, CMAP, retryable reads/writes, sessions, transactions, change streams, CSFLE, etc.). Spec test fixtures (YAML/JSON) are vendored under `test/spec/`. When behavior is ambiguous, the spec wins; do not change behavior away from the spec without raising it there first.
- **[mongodb/js-bson](https://github.com/mongodb/js-bson)** — BSON serialization (`bson` npm package). Owned by this team. Bug reports and changes that touch BSON encoding/decoding belong there.
- **[mongodb-js/kerberos](https://github.com/mongodb-js/kerberos)** — Native Kerberos bindings (`kerberos` npm package, optional dependency). Owned by this team. GSSAPI auth issues are typically fixed there, not in the driver.
- **[mongodb-js/mongodb-client-encryption](https://github.com/mongodb-js/mongodb-client-encryption)** — Native bindings to `libmongocrypt` for CSFLE/Queryable Encryption (`mongodb-client-encryption` npm package). Owned by this team.
- **[mongodb-js/zstd](https://github.com/mongodb-js/zstd)** and **[mongodb-js/saslprep](https://github.com/mongodb-js/saslprep)** — Compression and SCRAM SASLprep helpers, also team-owned.

**Do not hand-edit**: `lib/` (build output), `mongodb.d.ts` (generated), `HISTORY.md` (release-please managed), `test/spec/` (vendored from specifications repo).

## Common Commands

### Building

```bash
npm run build:ts          # Compile TypeScript to lib/
npm run check:ts          # Type-check without emitting
```

### Linting

```bash
npm run check:eslint      # Run ESLint
npm run fix:eslint        # Auto-fix ESLint issues
```

### Testing

Integration tests require a running MongoDB instance (unit tests do not). To start one locally:

```bash
git submodule update --init
export DRIVERS_TOOLS=$(pwd)/drivers-evergreen-tools
VERSION='latest' TOPOLOGY='replica_set' bash .evergreen/run-orchestration.sh
source mo-expansion.sh
```

```bash
npm run check:unit        # Unit tests (no database required)
npm run check:test        # Integration tests (requires database)
npm test                  # Full check (lint + d.ts/tsd) + unit + integration

# Run a single test by name pattern
npm run check:unit -- -g "pattern"
npm run check:test -- -g "pattern"
```

Tests use Mocha with 60-second timeout. Integration tests use a custom metadata UI that supports test filtering by topology, MongoDB version, auth, etc. via metadata:

```js
describe(
  'my test',
  { metadata: { requires: { topology: ['replicaset'], mongodb: '>=6.0' } } },
  function () {}
);
```

## Architecture

### Layered Design

```
Public API (MongoClient, Db, Collection, Cursors)
  → Operations (CRUD, Aggregation, Indexes, Bulk writes)
    → Sessions & Transactions
      → SDAM – Server Discovery And Monitoring (src/sdam/)
        → CMAP – Connection Management And Pooling (src/cmap/)
          → Wire Protocol & BSON serialization
```

### Key Source Directories

- **`src/operations/`** — Each database command is an `AbstractOperation` subclass. Operations declare aspects (retryable, read/write, explainable) via Symbols. `execute_operation.ts` is the central execution engine handling retries, sessions, server selection.
- **`src/sdam/`** — Topology discovery and monitoring. `topology.ts` manages servers, `server_selection.ts` picks the best server based on read preference and latency, `monitor.ts` sends periodic heartbeats.
- **`src/cmap/`** — Connection pooling per server, wire protocol encoding/decoding, authentication handshakes. `auth/` contains implementations for each auth mechanism (SCRAM, X.509, AWS, OIDC, Kerberos, PLAIN).
- **`src/cursor/`** — `AbstractCursor` base with lazy evaluation, async iteration, and streaming. Specialized cursors: `FindCursor`, `AggregationCursor`, `ChangeStreamCursor`, etc.
- **`src/bulk/`** — Ordered and unordered bulk write operations.
- **`src/client-side-encryption/`** — Auto-encryption and explicit encryption (CSFLE/Queryable Encryption).
- **`src/gridfs/`** — GridFS file storage using upload/download streams.

### How Operations Execute

1. User calls a method (e.g., `collection.insertOne()`)
2. An operation object is created (e.g., `InsertOperation`)
3. `executeOperation()` handles: implicit session creation → server selection → connection checkout → command building → wire protocol send → response handling → retry on transient errors
4. Connection returned to pool, session cleaned up

### Test Structure

- **`test/unit/`** — Mirrors `src/` structure. No database interaction, uses mocks.
- **`test/integration/`** — Real database tests organized by feature area.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mongodb/node-mongodb-native](https://github.com/mongodb/node-mongodb-native) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
