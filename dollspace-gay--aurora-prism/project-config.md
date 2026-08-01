---
trigger: always_on
description: This document explains the background "agents" that power the AT Protocol App View: how data flows from the Bluesky network into your database, how processing is distributed, and how to configure, operate, and extend these workers.
---

## Agents and Background Services

This document explains the background "agents" that power the AT Protocol App View: how data flows from the Bluesky network into your database, how processing is distributed, and how to configure, operate, and extend these workers.

### TL;DR
- **Ingest**: Worker 0 connects to the relay via `firehose` and writes events to Redis Streams.
- **Distribute**: All workers run parallel consumer pipelines that read from Redis and call the `eventProcessor`.
- **Persist**: The `eventProcessor` validates lexicons, enforces ordering, writes to PostgreSQL, and emits labels/notifications.
- **Backfill**: Optional historical import via `@atproto/sync` firehose backfill or full-repo CAR backfill.
- **Maintain**: Data pruning, DB health checks, metrics, and label streaming run continuously.

---

## Architecture Overview

- **App Server** (`server/index.ts`, `server/routes.ts`)
  - Boots Express, initializes Redis connections, starts WebSocket endpoints, and spins up agents.
  - Sets worker identity via `NODE_APP_INSTANCE`/`pm_id` and runs role-specific tasks.

- **Firehose Ingestion Agent** (`server/services/firehose.ts`)
  - Connects to the relay (`RELAY_URL`), handles keepalive ping/pong, stall detection, auto-reconnect, and cursor persistence to DB every 5s.
  - Only worker 0 connects to the relay and publishes events to Redis.

- **Redis Queue (Event Bus)** (`server/services/redis-queue.ts`)
  - Uses Redis Streams with a single stream (`firehose:events`) and consumer group (`firehose-processors`).
  - Buffers cluster metrics and exposes status/recents via keys (`cluster:metrics`, `firehose:status`, `firehose:recent_events`).
  - Provides dead-consumer recovery via periodic `XAUTOCLAIM`-like behavior (`claimPendingMessages`).

- **Consumer Pipelines (Workers)** (`server/routes.ts`)
  - All workers run 5 parallel pipelines each to consume from Redis (`consume(..., 300)`), process with `eventProcessor`, and `xack` on success.
  - Duplicate (`23505`) and FK race (`23503`) errors are treated as success to ensure idempotency.

- **Event Processor** (`server/services/event-processor.ts`)
  - Validates against lexicons, sanitizes content, writes posts/likes/reposts/follows/lists/etc., and creates notifications.
  - Maintains TTL queues for out-of-order events (e.g., like before post) and flushes once dependencies arrive.
  - Auto-resolves handles/DIDs on-demand and respects per-user collection opt-out (`dataCollectionForbidden`).

- **Backfill Agents**
  - `server/services/backfill.ts` (Firehose Backfill via `@atproto/sync`)
    - Historical replay with optional cutoff (`BACKFILL_DAYS`), dedicated DB pool, periodic progress save, batching, and backpressure.
  - `server/services/repo-backfill.ts` (Repo CAR Backfill via `@atproto/api`)
    - Full per-repo import from PDS, parses CAR, walks MST for real CIDs (synthetic fallback), concurrent fetches.

- **Maintenance Agents**
  - `server/services/data-pruning.ts`: Periodic deletion beyond `DATA_RETENTION_DAYS` (safety minimums and batch caps).
  - `server/services/database-health.ts`: Periodic DB connectivity/table existence/count checks and loss detection.
  - `server/services/metrics.ts` and `server/services/log-collector.ts`: In-memory metrics and rolling logs (also surfaced to the dashboard).
  - `server/services/instance-moderation.ts`: Operator-driven label application and policy transparency.

---

## Event Flow

1. Firehose connects to `RELAY_URL` and emits `#commit`, `#identity`, `#account` events.
2. Worker 0 serializes them into lightweight objects and pushes to Redis Stream `firehose:events`.
3. Every worker runs multiple pipelines that call `redisQueue.consume()` to fetch batches (blocking for ~100ms), then `processEvent` with `eventProcessor`.
4. On success, the message is acknowledged (`xack`); every ~5s each pipeline also claims abandoned messages.
5. `eventProcessor` performs:
   - User ensure/creation with handle resolution.
   - Validation (lexicon), sanitization, and writes to PostgreSQL.
   - Deferred queueing and later flush for out-of-order dependencies.
   - Label application and notification fanout.

Cursor persistence:
- Live firehose: worker 0 saves cursor to DB every 5 seconds (`firehoseCursor` table).
- Backfill: a dedicated runner updates progress (`saveBackfillProgress`), including counts and last update time.

---

## Agents in Detail

### Firehose Ingestion (`server/services/firehose.ts`)
- Keepalive (ping every 30s), pong timeout (45s), stall threshold (2m) with forced reconnect.
- Concurrency guard for commit handling (`MAX_CONCURRENT_OPS`) with queue backpressure and drop policy when overloaded.
- Status and recent events are mirrored into Redis for the dashboard.

### Redis Queue (`server/services/redis-queue.ts`)
- Stream: `firehose:events`, group: `firehose-processors`.
- `consume(consumerId, count)` uses `XREADGROUP` with short block; `ack(messageId)` acks processed entries.
- `claimPendingMessages(consumerId, idleMs)` reclaims abandoned messages for resilience.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dollspace-gay/Aurora-Prism](https://github.com/dollspace-gay/Aurora-Prism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
