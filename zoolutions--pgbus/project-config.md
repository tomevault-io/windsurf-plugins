---
trigger: always_on
description: PostgreSQL-native job processing and event bus for Rails, built on PGMQ.
---

# Pgbus

PostgreSQL-native job processing and event bus for Rails, built on PGMQ.

## Tech Stack

- **Ruby**: >= 3.3 | **Rails**: >= 7.1
- **Transport**: pgmq-ruby (PGMQ — extension or embedded SQL)
- **Concurrency**: concurrent-ruby
- **Autoloading**: zeitwerk
- **Testing**: RSpec
- **Linting**: RuboCop

## Critical Rules

### Never Do
1. **NO direct PGMQ calls** — always go through `Pgbus::Client`
2. **NO hardcoded queue names** — use `config.queue_name()`
3. **NO raw SQL in dashboard** — use `Web::DataSource`
4. **NO `Marshal.load`** — JSON serialization only
5. **NO unsynchronized shared state** — use Mutex or Concurrent primitives
6. **NO swallowing errors** — log via `Pgbus.logger`, track in `pgbus_failed_events`
7. **NO `Record` suffix on model classes** — see Model Naming below

### Always Do
1. **TDD**: Write tests BEFORE implementation
2. **Worker recycling**: Configure `max_jobs`, `max_memory_mb`, `max_lifetime`
3. **Dead letter routing**: Check `read_ct` > `max_retries`
4. **LISTEN/NOTIFY**: Use `enable_notify_insert` for instant wake-up
5. **Queue prefix**: All queues through `config.queue_name()`
6. **Visibility timeout**: Always pass `vt:` parameter on reads
7. **Performance**: Measure before/after for hot-path changes (`/perf`); see `docs/performance.md`

## Commands

```bash
bundle exec rspec          # Run tests
bundle exec rubocop        # Lint
bundle exec rake           # Both
bundle exec rake bench     # Unit benchmarks (serialization, client, executor)
bundle exec rake bench:one[client_bench]  # Single benchmark by name
bundle exec rake bench:memory             # Detailed memory profiling
bundle exec rake bench:integration        # Real DB benchmarks (requires PGBUS_DATABASE_URL)
bundle exec rake bench:streams            # SSE streaming benchmarks (requires PGBUS_DATABASE_URL)
```

## Slash Commands

| Command | Purpose |
|---------|---------|
| `/lfg` | Full autonomous workflow: branch → understand → explore → plan → TDD → verify → PR |
| `/plan` | Fable-powered planning → GitHub issue or `docs/plans/` markdown (read-only; execute with `/lfg`) |
| `/github-review-comments` | Process unresolved PR review comments |
| `/review-pr` | Review a PR for pattern compliance |
| `/tdd` | Enforce RED → GREEN → REFACTOR cycle |
| `/security` | Security audit (PGMQ ops, connections, auth, deserialization) |
| `/architect` | Coordinate multi-layer development |
| `/perf` | Benchmark current branch against main (before/after with worktree) |

Commands and agents pin a model tier via frontmatter aliases: `sonnet` for pattern-following implementation (the default), `opus` for orchestration, security, and full PR review, `fable` for read-only planning (`/plan`). Use aliases, not full model IDs, so commands track the latest model in each tier. When spawning subagents for mechanical work (file finding, pattern scans), pass a cheaper model explicitly rather than letting them inherit the session model.

## Architecture

```
Layer 6: Dashboard       app/controllers/pgbus/, app/views/pgbus/
Layer 5: CLI             lib/pgbus/cli.rb
Layer 4: Process Model   lib/pgbus/process/ (supervisor, worker, dispatcher, consumer)
         Execution Pools lib/pgbus/execution_pools/ (thread_pool, async_pool)
Layer 3: Event Bus       lib/pgbus/event_bus/ (publisher, subscriber, registry, handler)
Layer 2: ActiveJob       lib/pgbus/active_job/ (adapter, executor)
Layer 1: Client          lib/pgbus/client.rb (PGMQ wrapper)
Layer 0: Config          lib/pgbus/configuration.rb, config_loader.rb
```

## Model Naming

ActiveRecord models live in `app/models/pgbus/` and inherit from `Pgbus::ApplicationRecord`.
**Never use a `Record` suffix.** Resolve naming conflicts as follows:

| Model Class | Table | Why not the obvious name |
|---|---|---|
| `Pgbus::BusRecord` | (abstract) | Base class in `lib/pgbus/` — loaded by Zeitwerk gem loader, avoids engine boot-order issues |
| `Pgbus::ApplicationRecord` | (abstract) | Backward-compatible alias for `BusRecord` in `app/models/` |
| `Pgbus::BatchEntry` | `pgbus_batches` | `Pgbus::Batch` is the batch API class |
| `Pgbus::BlockedExecution` | `pgbus_blocked_executions` | — |
| `Pgbus::ProcessEntry` | `pgbus_processes` | `Process` conflicts with Ruby's `Process` module |
| `Pgbus::ProcessedEvent` | `pgbus_processed_events` | — |
| `Pgbus::RecurringExecution` | `pgbus_recurring_executions` | — |
| `Pgbus::RecurringTask` | `pgbus_recurring_tasks` | `Pgbus::Recurring::Task` is a different namespace |
| `Pgbus::Semaphore` | `pgbus_semaphores` | `Pgbus::Concurrency::Semaphore` is a different namespace |

When a model name collides with a service/module name, prefer `Entry` suffix or a descriptive alternative over `Record`.

## Separate Database Support

Pgbus supports running in the primary database or a dedicated database (like SolidQueue).

**Configuration** (`config.connects_to`):
- `nil` (default) — uses the primary Rails database
- `{ database: { writing: :pgbus } }` — uses a separate database

**Generator flags**:
- `rails generate pgbus:install --database=pgbus` — migrations go to `db/pgbus_migrate/`
- `rails generate pgbus:add_recurring --database=pgbus` — recurring migrations also go to `db/pgbus_migrate/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zoolutions/pgbus](https://github.com/zoolutions/pgbus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
