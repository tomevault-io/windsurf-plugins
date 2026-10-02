---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this gem is

Optional Postgres LISTEN/NOTIFY wake-ups for Solid Queue: a database trigger on `solid_queue_ready_executions` fires `pg_notify(channel, queue_name)` on commit, and one listener thread per worker process wakes matching workers via Solid Queue's public `wake_up`. Polling remains the correctness backstop — notifications are purely a latency optimization, and **every failure mode must degrade loudly to stock Solid Queue behavior** (banner in the log, gem inactive, polling intervals untouched or restored).

Two hard constraints shape everything:

1. **Zero monkey patches.** Solid Queue is integrated only through documented lifecycle hooks (`on_worker_start`/`on_worker_stop`, wired in the Railtie inside `ActiveSupport.on_load(:solid_queue)`) and public methods (`wake_up`, `queues`, `polling_interval=`, `pool.idle?`, `alive?`). Some of these are public-by-visibility rather than documented contract — that is why CI has an allowed-to-fail cell against `solid_queue@main`.
2. **The gem must load and pass its unit tier with no `pg`, `rails`, or `solid_queue` loaded.** Every reference to those is lazy (inside method bodies, `defined?`-guarded, error classes resolved by name at rescue time in `Listener.connection_errors`). Related trap: inside `module SolidQueue`, bare `Process` resolves to the `SolidQueue::Process` AR model once solid_queue is loaded — always write `::Process`.

## Commands

Postgres is required for integration tests (LISTEN can't be faked). Either `docker compose up -d` (publishes 127.0.0.1:55432, user `postgres` — what CI uses) or point env vars at a local server:

```bash
POSTGRES_PORT=5432 POSTGRES_USER=$(whoami) bundle exec rake test   # full suite (unit + integration)
bundle exec rake test:unit                                         # no DB, no Rails — runs anywhere
POSTGRES_PORT=5432 POSTGRES_USER=$(whoami) bundle exec rake test:integration
bundle exec rubocop                                                # rubocop-rails-omakase

# Single file / single test:
bundle exec ruby -Itest test/unit/queue_matcher_test.rb
POSTGRES_PORT=5432 POSTGRES_USER=$(whoami) bundle exec ruby -Itest test/integration/wake_latency_test.rb -n /latency/

# Other Rails versions (Appraisals: rails_7_1, rails_8_1, solid_queue_main):
BUNDLE_GEMFILE=gemfiles/rails_7_1.gemfile bundle exec rake test
bundle exec appraisal generate                                     # after editing Appraisals
```

`test/integration_helper.rb` creates and schema-loads both dummy databases itself on every run — there is no separate db:setup step, and a stale test DB can never explain a failure.

## Architecture

`lib/solid_queue/listen_notify.rb` is the facade: config (`mattr_accessor`, mirroring solid_queue's style; copied from `config.solid_queue_listen_notify` by the Railtie), `register`/`deregister` (called from the lifecycle hooks; fully rescued — this gem must never make a worker look broken), the per-pid memoized `operational?`, and `after_fork`.

Flow at worker boot: `register(worker)` → `operational?` runs **Preflight** once per process → if operational, raise the worker's `polling_interval` to `fallback_polling_interval` (only ever raise) → `Registry.instance.register(worker, restore_interval:)` records the replaced interval with the worker and starts the **Listener** on first registration. The registry is the single owner of per-worker state (queue snapshot + replaced interval); on a fatal listener death it hands the `[worker, original_interval]` pairs back to the facade, which restores and wakes them.

- **`Preflight`** (`preflight.rb`) — decides operational-or-not; never raises; each failure prints an actionable WARN/ERROR banner (loud failure is a product requirement, not a nicety). Checks in order: enabled → channel 1–63 bytes → adapter is PG → trigger installed (auto-installed by default when missing) → trigger function body contains the exact `pg_notify('<channel>', ...)` call → **cross-connection self-test**: LISTEN on the listener's connection path, NOTIFY from a pooled queue-DB connection. Cross-connection is what detects both PgBouncer transaction pooling and a `listen_database` pointing at the wrong database.
- **`Registry`** (`registry.rb`) — per-process singleton multiplexing one listener to N workers (async mode runs N workers as threads in one process). Locking discipline is documented in its header comment and must be preserved: the mutex is never held across `wake_up`, `listener.stop`, or anything blocking; listeners are detached under the lock and stopped outside it. Cleanup is crash-tolerant, never hook-dependent (`on_worker_stop` is skipped on SIGQUIT `exit!`): dead workers are reaped on register and on the keepalive tick.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cmer/solid_queue-listen_notify](https://github.com/cmer/solid_queue-listen_notify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
