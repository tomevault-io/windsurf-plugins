---
trigger: always_on
description: > A type-safe event sourcing toolkit for Go. This document maps every public API surface (excluding `workflow/`).
---

# Chronicle — Developer Map

> A type-safe event sourcing toolkit for Go. This document maps every public API surface (excluding `workflow/`).

## Architecture at a glance

```
Your Aggregate (embed aggregate.Base)
        │
        ▼
  aggregate / chronicle     ← Repositories, Root, Snapshotter
        │
        ▼
      event                 ← Interfaces: Log, Transformer, Registry, Projections
        │
        ▼
      version               ← Version, Selector, ConflictError
```

`eventlog/` and `snapshotstore/` are pluggable implementations of the interfaces above.

---

## How the pieces fit together

### The event log is built from small interfaces

The lowest layer is two tiny interfaces: `Reader` (read a single stream) and `Appender` (write with an optimistic concurrency check). `Log` is just `Reader + Appender` — the minimum viable event log. Everything else is opt-in:

```
Reader + Appender = Log                       ← bare minimum
Log    + GlobalReader = GlobalLog             ← adds cross-stream reads (for projections)
TransactionalLog + Transactor = TransactionalEventLog  ← adds atomic TX support
```

This means a custom backend only needs to implement what it actually supports. NATS implements `Log` + `TransactionalEventLog` but not `GlobalReader` (no global ordering). Postgres and SQLite implement everything. Memory implements everything plus `DeleterLog`.

### Transactions are opt-in

A plain `Log` has no concept of transactions — `AppendEvents` is a single atomic write. If your backend supports real transactions (Postgres, SQLite), it can also implement `TransactionalEventLog[TX]`, which exposes `AppendInTx` and `WithinTx`. This unlocks two things:

1. **`TransactionalRepository`** — saves events + runs a processor (outbox, projection update) in a single TX.
2. **`TransactableLog`** — wraps a transactional log + an optional `SyncProjection`, so the projection runs inside the same TX as the append.

If you don't need transactions, use a plain `ESRepo` with a plain `Log`. No extra complexity.

### Repositories are stackable decorators

The base is `ESRepo` — it loads by replaying events and saves by appending. On top of that, decorators add capabilities:

```
ESRepo                           ← replay all events on load, append on save
  └─ ESRepoWithSnapshots         ← loads from snapshot first, replays only new events
      └─ ESRepoWithRetry         ← retries Save on ConflictError with backoff
```

Each decorator implements the same `Repository` interface, so they compose freely. `FusedRepo` exists for cases where you want to override just one method (e.g. only `Save`) without re-implementing everything.

### Projections: sync vs async

**Sync projections** (`SyncProjection[TX]`) run inside the same transaction that appends events. They see the same TX handle, so updates to your read model are atomic with the event write. Use case: transactional outbox, strongly consistent denormalized tables.

**Async projections** (`AsyncProjection`) poll or tail the global event stream independently. They track progress via a `Checkpointer` and are managed by an `AsyncProjectionRunner`. Use case: eventually consistent read models, cross-service integration, analytics.

### Transformers are a pipeline

Transformers sit between your domain events and the event log. On write, they run in order (e.g. encrypt → compress). On read, they run in reverse (decompress → decrypt). This is the hook for:

- **Encryption / crypto shredding** — encrypt sensitive fields before storage, decrypt on load.
- **Event upcasting** — transform old event shapes into new ones on read (1-to-many supported).
- **Cross-cutting concerns** — `AnyTransformer` works across all event types, adapted per-repo via `AnyTransformerToTyped`.

### The registry connects event names to Go types

When events are stored, only the name (`EventName() string`) and encoded bytes are persisted. The `Registry` maps names back to factory functions (`func() E`) so the framework can decode bytes into concrete Go types. Repos auto-register on creation by default. For cross-aggregate uniqueness, use a shared `AnyEventRegistry`.

---

## `chronicle` (root package)

Convenience re-exports. You wire everything from here.

| Function | What it does |
|---|---|
| `NewEventSourcedRepository(log, createRoot, transformers, ...opts)` | Standard ES repo. JSON by default, auto-registers events. |
| `NewEventSourcedRepositoryWithSnapshots(repo, snapStore, snapshotter, policy, ...opts)` | Wraps a repo with snapshot load/save. |
| `NewEventSourcedRepositoryWithRetry(repo, ...retryOpts)` | Wraps a repo with automatic retry on `ConflictError`. Defaults to 3 attempts. |
| `NewTransactionalRepository(txLog, createRoot, transformers, processor, ...opts)` | Atomic save + processor in one TX. For outbox / sync projections. |
| `NewTransactionalRepositoryWithTransactor(transactor, txLog, createRoot, transformers, processor, ...opts)` | Same, but decouples the transactor from the log. |
| `NewEventRegistry[E]()` | Typed event registry. |
| `NewAnyEventRegistry()` | Type-erased registry for cross-aggregate uniqueness. |

---

## `event/`

### Core types

| Type | Role |
|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DeluxeOwl/chronicle](https://github.com/DeluxeOwl/chronicle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
