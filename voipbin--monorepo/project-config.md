---
trigger: always_on
description: Platform-internal cron scheduler (VOIP-1283). Absorbs external cron surfaces (the number-manager Kubernetes CronJob, future ticker migrations) into one DB-driven dispatch engine with an audit trail, at-most-once claim semantics, and Prometheus visibility. Phase 1 schedules are all platform-owned (nil customer); there is no user-visible API surface yet.
---

# bin-schedule-manager

Platform-internal cron scheduler (VOIP-1283). Absorbs external cron surfaces (the number-manager Kubernetes CronJob, future ticker migrations) into one DB-driven dispatch engine with an audit trail, at-most-once claim semantics, and Prometheus visibility. Phase 1 schedules are all platform-owned (nil customer); there is no user-visible API surface yet.

> Cross-cutting rules (verification workflow, branch/commit format, worktrees, Alembic, RST sync) live in the root [CLAUDE.md](../CLAUDE.md). This file covers only what is specific to `bin-schedule-manager`.

## Key facts

- **MySQL** tables `schedule_schedules` / `schedule_executions` (audit trail), **Redis** claim locks via redsync (`schedule:lock:<id>`).
- **RabbitMQ queue**: `bin-manager.schedule-manager.request`
- **Subscribes to**: `bin-manager.customer-manager.event` (`customer_deleted` → delete the customer's schedules)
- **Publishes**: internal events only on `bin-manager.schedule-manager.event` (`schedule_created/updated/deleted`, `execution_succeeded/failed`). **No customer webhooks in Phase 1** — every schedule is nil-customer, so there is deliberately no `models/schedule/webhook.go` and no RST struct docs (Phase 3 scope).
- **replicas: 2** by design — every replica runs the same tick loop; Redis lock + DB CAS claim make concurrent replicas safe (kill -9 failover without double-fire).
- **Housekeeping dogfoods the engine**: `execution-retention` and `database-backup` are seeded schedules that target the service's own request queue (`/v1/executions/prune`, `/v1/backups`).

## Package layout

| Package | Role |
|---------|------|
| `cmd/schedule-manager` | Daemon entry point (cobra; Bootstrap/LoadGlobalConfig) |
| `cmd/schedule-control` | Admin CLI (direct DB/cache, no RabbitMQ — works when the broker path is unhealthy) |
| `internal/config` | Viper/pflag config binding |
| `models/schedule` | Schedule struct, cron helpers, filters, internal event types |
| `models/execution` | Execution struct (audit row), status machine, filters |
| `pkg/listenhandler` | RabbitMQ RPC router (regex dispatch) |
| `pkg/subscribehandler` | Event consumer (`customer_deleted`) |
| `pkg/schedulehandler` | Schedule CRUD, validation, next-run computation, name uniqueness |
| `pkg/dispatchhandler` | Tick loop, claim (lock + CAS), dispatch, record, reaper, manual execute |
| `pkg/backuphandler` | mysqldump subprocess + gzip + retention pruning (design §7) |
| `pkg/dbhandler` | MySQL via squirrel; sqlite test harness |
| `pkg/cachehandler` | Redis + redsync claim locks |

## Request routing

| Pattern | Operations |
|---------|-----------|
| `/v1/schedules$` | POST (create; validates cron, method whitelist, `target_queue` allowlist, active-name uniqueness) |
| `/v1/schedules?(.*)$` | GET (list with filters/pagination) |
| `/v1/schedules/<uuid>$` | GET, PUT (cron change ⇒ `tm_next_run=NULL`), DELETE (soft) |
| `/v1/schedules/<uuid>/execute$` | POST (manual fire-now; never consumes the cron slot) |
| `/v1/executions?(.*)$` | GET (audit trail) |
| `/v1/executions/prune$` | POST (internal; invoked by the `execution-retention` schedule) |
| `/v1/backups$` | POST (internal; invoked by the `database-backup` schedule) |

## schedule-control CLI

Direct DB/cache access, no RabbitMQ — `schedule disable` stops a misbehaving dispatch loop even when the broker is down. Name arguments resolve in the platform (nil-customer) namespace; UUIDs resolve by id.

```bash
./bin/schedule-control schedule list
./bin/schedule-control schedule get number-renew
./bin/schedule-control schedule disable number-renew
./bin/schedule-control schedule enable number-renew
./bin/schedule-control execution list --schedule-id <uuid>
```

## Common commands

```bash
go build -o ./bin/ ./cmd/...
go test ./...
go generate ./...
golangci-lint run -v --timeout 5m
```

## Configuration

| Env | Description | Default |
|-----|-------------|---------|
| `DATABASE_DSN` | MySQL DSN | required |
| `RABBITMQ_ADDRESS` | RabbitMQ server | required |
| `REDIS_ADDRESS` | Redis server | required |
| `REDIS_PASSWORD` | Redis auth | empty |
| `REDIS_DATABASE` | Redis DB index | `1` |
| `PROMETHEUS_ENDPOINT` | Metrics path | `/metrics` |
| `PROMETHEUS_LISTEN_ADDRESS` | Metrics listen address | `:2112` |
| `SCHEDULE_TICK_INTERVAL_SEC` | Dispatch loop scan cadence (seconds) | `10` |
| `SCHEDULE_DISPATCH_CONCURRENCY` | Max in-flight dispatches per replica | `10` |
| `SCHEDULE_EXECUTION_RETENTION_DAYS` | Execution-row retention applied by `/v1/executions/prune` | `90` |
| `SCHEDULE_BACKUP_DIR` | Backup dump directory. **No default — deliberately.** A default would let a surface that forgot to mount the shared volume "succeed" into ephemeral container disk (silent backup loss). Unset ⇒ the backup job fails loudly; nothing else breaks. | (none) |
| `SCHEDULE_BACKUP_RETENTION_COUNT` | Newest backup files to keep | `7` |

## CRITICAL: Runtime image deviation (debian-slim + mysql-community-client)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [voipbin/monorepo](https://github.com/voipbin/monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
