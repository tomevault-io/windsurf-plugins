---
trigger: always_on
description: SubmitQueue is a distributed system for managing code submission workflows. It follows clean architecture with interface-driven extensibility.
---

# SubmitQueue Repository Guide

## Key Concepts

SubmitQueue is a distributed system for managing code submission workflows. It follows clean architecture with interface-driven extensibility.

**Immutability and Eventual Consistency:**

1. **Immutable entities** — once created, don't modify in place. Create new versions with updated fields.
2. **Eventual consistency** — handle stale reads, idempotent operations, and convergence over time.
3. **Event sourcing** — store events (what happened) rather than just current state for critical changes.
4. **Optimistic locking** — use version numbers instead of pessimistic locks. Avoid transactions; prefer optimistic concurrency and retries. **Version arithmetic lives in the controller, not the storage layer.** Update methods take both `oldVersion` (the where-clause guard) and `newVersion` (the value to write); the store performs a pure conditional write. Controllers compute `newVersion = oldVersion + 1`, call the store, and only assign `entity.Version = newVersion` after the call succeeds. Pre-incrementing in memory before the call is a bug pattern — on error the in-memory version drifts ahead of the database. See [submitqueue/extension/storage/README.md](submitqueue/extension/storage/README.md).
5. **Idempotency keys** — include unique request IDs, check for duplicates before executing.

```go
// Immutable entity pattern
type Request struct {
    ID        string
    Version   int       // For optimistic locking
    Status    Status
    CreatedAt int64
    UpdatedAt int64
}

// Controller pattern — version arithmetic outside storage, assigned only on success
newVersion := request.Version + 1
if err := store.UpdateStatus(ctx, request.ID, request.Version, newVersion, newStatus); err != nil {
    return err
}
request.Version = newVersion
```

## Architecture

### Project Layout

```
submitqueue/                        # repo root (Go module github.com/uber/submitqueue)
├── api/                            # Published wire contracts (cross-domain/external)
│   ├── submitqueue/{gateway,orchestrator}/{proto,protopb}/   # RPC (proto)
│   ├── stovepipe/{proto,protopb}/  # single-service RPC (proto) — no service segment yet
│   ├── runway/{proto,protopb}/     # RPC (proto) — single-service domain, no service segment
│   └── runway/messagequeue/        # external queue contracts (proto + protojson)
├── platform/                       # SHARED cross-domain packages — no domain deps
│   ├── errs/, metrics/, consumer/, http/
│   ├── base/                       # SHARED entities (change/, messagequeue/, …)
│   └── extension/                  # SHARED extension contracts + backends (counter/, messagequeue/, …)
├── submitqueue/                    # SubmitQueue domain
│   ├── gateway/                    # Gateway service (port 8081) - entry point
│   ├── orchestrator/               # Orchestrator service (port 8082) - coordinates jobs
│   ├── entity/                     # SubmitQueue-specific domain entities
│   ├── extension/                  # SubmitQueue-specific extension impls (storage, counter, mergechecker, …)
│   └── core/                       # SubmitQueue-internal shared infra (consumer wiring, request, topickey, …)
├── stovepipe/                      # Stovepipe domain (single Ping-only service for now)
│   └── controller/                 # Business logic (currently just Ping); entity/extension/core added as it grows
├── runway/                         # Runway domain (single service — the domain *is* the service)
│   └── controller/                 # Runway service controllers (consumes the merge queues; no gateway/orchestrator split)
├── tool/                           # Development and CI tooling
├── service/                        # Runnable server/client wiring (entry points + Docker Compose)
│   ├── submitqueue/                # Runnable SubmitQueue servers/clients + Docker Compose
│   ├── stovepipe/                  # Runnable Stovepipe server/client + Docker Compose
│   └── runway/                     # Runnable Runway server/client + Docker Compose
├── test/
│   ├── e2e/submitqueue/            # End-to-end tests (full stack)
│   ├── integration/                # Integration tests (platform/, submitqueue/, stovepipe/, …)
│   └── testutil/                   # Test utilities (ComposeStack, MySQL helpers)
└── doc/                            # Documentation
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uber/submitqueue](https://github.com/uber/submitqueue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
