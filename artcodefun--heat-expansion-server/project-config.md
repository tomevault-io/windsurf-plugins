---
trigger: always_on
description: This repo is a Go backend for the Heat Expansion strategy game. It uses Hexagonal Architecture, DDD, and CQRS.
---

# Heat Expansion Server

This repo is a Go backend for the Heat Expansion strategy game. It uses Hexagonal Architecture, DDD, and CQRS.

## Architecture & Layout

This repo is a **modular monolith**: each service lives under `internal/<service>`.

**Game service (primary):** `internal/game`
- `domain/`: Aggregates and domain logic (e.g. `UserBase`, `MilitaryOperation`, events, value objects).
- `application/`: application contracts, CQRS handlers, ports, and services
  - `commands/`: Write-side command handlers.
  - `queries/`: Read-side query handlers.
  - `readmodels/`: Query-side models returned by application queries.
  - `ports/`: Interfaces for repositories, schedulers, token providers, event publishing, transactions, etc.
  - `services/`: App-level services (access control, provisioning, outbox loop, ...).
- `infrastructure/`: Secondary adapters (DB/sqlc, readstore, jobs, events, security, content, ...).
- `interfaces/http`: Primary adapter (HTTP handlers/DTOs/middleware/router).
- `bootstrap/`: Dependency wiring for the game service.

**Auth service:** `internal/auth` (IAM, JWT, integration producers)

**Billing service:** `internal/billing` — crystal-package purchases via YooKassa. The webhook handler never trusts the request body; it re-queries YooKassa for canonical payment state. On success it emits `CrystalsPurchasedV1`, which the game service consumes to credit crystals (idempotent on `order_id`).

**Admin service:** `internal/admin` — back-office API for operators to manage game content and billing configuration. Uses server-side sessions with opaque bearer tokens (not JWT). Content management operations (prototypes, translations, crystal packages) are proxied to the game and billing modules via gRPC. Port model types used for gRPC proxying are co-located with their port interfaces in `ports/` rather than treated as CQRS read models. Owns its own database schema (`admin.admins`, `admin.sessions`).

**Shared contracts:** `contracts/` (versioned integration event schemas and HTTP OpenAPI contracts)

**Shared platform:** `internal/platform/` — infrastructure adapters reused across services (RabbitMQ publisher/consumer, JWT token validator, in-process event publisher, i18n translator core). When an adapter is needed by more than one service, it lives here rather than being duplicated.

## Key Patterns & Conventions

The patterns below apply to the **Game** service (`internal/game`) unless stated otherwise.

### Commands vs. queries
- Commands mutate state and live in `internal/game/application/commands`. They run inside `TransactionManager.WithTx`, use repositories from `application/ports`, and interact with aggregates in `domain/`.
- Queries do not mutate state and live in `internal/game/application/queries`. They depend on read repositories from `internal/game/infrastructure/readstore` and use shared services like access control from `application/services`.

### Repositories & transactions
- Repositories are declared as ports (e.g. `UserBaseRepository`, `SectorRepository`) and implemented in `internal/game/infrastructure/db/repo` using sqlc-generated `gen` packages.
- Use `Tx(tx)` on repositories and outbox interfaces when working inside a transaction; do not create new DB connections directly in core or handlers.
- Repository lookups return `*domain.X` / `[]*domain.X`, never values — a zero-value struct on the error path is a plausible-but-invalid object that can leak into the core, whereas `nil` cannot. On a non-nil error the pointer is `nil` (missing rows return `nil, ErrNotFound`); callers check the error first and never dereference on the error path.

### Domain events & outbox
- Aggregates emit domain events via `EventProducer` in `internal/game/domain`.
- Command handlers do **not** publish directly; they call `OutboxEventRepository.Save(events)` inside the transaction.
- `OutboxService` (in `internal/game/application/services/outbox_service.go`) runs in a background loop from `App.Run` and pulls from the `domain_events` table to publish via `EventPublisher`.
- When adding new events, update outbox DTOs/mappers in `internal/game/infrastructure/db/dtos` and `mappers` rather than encoding from domain types directly in handlers.

### Scheduled jobs
- Jobs are created via `Scheduler.Schedule()` inside event handlers, which run outside the outbox transaction. This means job creation and event acknowledgement are not atomic — the system provides **at-least-once** delivery, not exactly-once. Duplicate job executions are possible under crash/retry scenarios.
- **Every job handler must be idempotent.** Re-executing a handler with the same input must be safe. Use domain state guards (e.g. check current phase/status before acting) so that a duplicate invocation is a no-op.
- Jobs are never created directly in command handlers or outside of event handlers, to keep the creation path predictable.
- Self-rescheduling jobs (jobs that schedule the next iteration of themselves) must use **positive-only jitter**. Never subtract from the period when computing the next `executeAt` — firing before the full period has elapsed can interact badly with time-based idempotency checks and cause legitimate runs to be silently skipped.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [artcodefun/heat-expansion-server](https://github.com/artcodefun/heat-expansion-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
