---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Wurk

100% API-compatible drop-in replacement for Sidekiq + Sidekiq Pro + Sidekiq Enterprise. Free. Fork-based swarm for multi-core parallelism. Mountable Rails engine.

Three pillars, all must stay true:

1. **100% drop-in.** Same Redis key schema, same job JSON, same Ruby DSL. Existing Sidekiq jobs and Redis data keep working on a one-line gem swap. Third-party gems (sidekiq-cron, sidekiq-unique-jobs, sidekiq-scheduler, sidekiq-status, sidekiq-failures, sidekiq-throttled, etc.) pass their own test suites against Wurk.
2. **Free.** Pro + Ent feature parity in the same gem. No tiers, no flags gating Ent behavior, no license checks.
3. **Measured.** Two suites, different jobs. `rake bench` is the REGRESSION gate (wurk vs its own past self; >5% on enqueue / fetch+execute / bulk enqueue / swarm boot / memory blocks merge). `rake bench:vs_sidekiq` is the COMPARISON vs stock Sidekiq. A green gate says nothing about Sidekiq. Wurk is currently SLOWER than stock Sidekiq (~0.45x-0.86x, see `docs/benchmarks.md`) — do not add a "faster" claim to the README, site, or llms.txt until that doc's numbers support it.

## Commands

| Task | Command |
|---|---|
| Install | `bundle install` |
| Full test suite (parallel) | `bin/rake test` |
| Single file | `bin/rake test TEST=test/path/to/file_test.rb` |
| Single test by name | `bin/rake test TEST=test/foo_test.rb TESTOPTS="--name=/pattern/"` |
| Parity tests (lifted from Sidekiq) | `bin/rake test:parity` |
| Ecosystem compat | `bin/rake test:ecosystem` |
| Benchmarks | `bin/rake bench` |
| Dummy app | `cd test/dummy && bin/rails s` |
| Standalone runner | `exe/wurk` |
| Dashboard build | `bin/rake frontend:build` (`bun install` + `vite build` → `vendor/assets/`) |
| Dashboard tests | `bun run test` in `frontend/` (vitest) |
| Dashboard HMR dev | `WURK_VITE_DEV=1` then boot dummy |
| Release | `bin/rake release` (Vite build → `vendor/assets/` → gem build → push) |

`WURK_DISABLED=1`, Rails console, and Rails test env all skip the railtie's auto-fork.

## Architecture

Layers and ownership — one reason to change per class. Don't blur these.

| Layer | Owns | Path |
|---|---|---|
| Engine | Dashboard mount, asset path, railtie | `lib/wurk/engine.rb`, `app/` |
| Railtie | `after_initialize` hook that starts the swarm | `lib/wurk/railtie.rb` |
| Swarm | Parent process; forks N children, PID supervision, rolling restart | `lib/wurk/swarm.rb` |
| Manager | Inside each child: thread pool, lifecycle, heartbeat | `lib/wurk/manager.rb` |
| Fetcher | BLMOVE reliable fetch: main queue → per-process private list | `lib/wurk/fetcher.rb` |
| Processor | Pops private list, runs middleware chain, invokes perform | `lib/wurk/processor.rb` |
| Client | Enqueue, Lua bulk path, Redis-outage local buffer | `lib/wurk/client.rb` |
| Middleware | Client + server chains (Sidekiq contract) | `lib/wurk/middleware/` |
| Web | Rack app serving the precompiled SolidJS SPA + JSON APIs | `lib/wurk/web.rb`, `app/` |
| RedisPool | Per-process pool over redis-client | `lib/wurk/redis_pool.rb` |

User-facing code (mount, controllers, views, generators, assets) lives in the engine. Non-user-facing (swarm, fetcher, processor, client, middleware) lives in plain Ruby under `lib/`. **Standalone mode must run without loading the engine.**

Sidekiq aliases — every public `Wurk::*` class is exposed under its `Sidekiq::*` name (`Sidekiq::Worker`, `Sidekiq::Batch`, `Sidekiq.configure_server`, `Sidekiq::Limiter`, …). The alias is the drop-in contract. Never break it.

## Boot ordering (do not reorder)

1. Host app boots fully; initializers run; eager-loaded constants resolved.
2. Railtie `after_initialize` fires.
3. Swarm closes parent-side connections that must not survive fork (DB, Redis).
4. Swarm forks N children.
5. Each child reconnects DB and opens a fresh Redis pool, then starts fetching.
6. Parent enters supervision loop.

Skip step 3 → leaked sockets in children. Skip step 5 → children corrupt each other's responses on a shared socket.

## Signals

| Signal | Target | Effect |
|---|---|---|
| SIGTERM / SIGINT | parent | Graceful drain. Relayed to children; in-flight finishes to `shutdown_timeout`; exit |
| SIGTSTP | parent | Quiet globally: relayed to children, each stops fetching; in-flight continues. One-way (matches Sidekiq TSTP, spec §21.3) — no resume; TERM to shut down |
| SIGUSR1 | parent | Rolling restart: fork replacement → wait for heartbeat → SIGTERM old slot → next |
| SIGUSR2 | child | Reopen log files (logrotate) |
| SIGKILL | any | Safe — private-list entries stay in Redis and are reclaimed on next boot |

## Conventions

- **SOLID, especially SRP.** Manager owns lifecycle; Fetcher owns Redis pop; Processor owns middleware+perform; Client owns enqueue.
- **Wire-compat is sacred.** Never change a Redis key, JSON field, or sorted-set score format. If a perf optimization would break compat, drop the optimization.
- **Frozen string literals everywhere.** Hot-loop allocations matter.
- **Per-fork Redis pool.** Never share a socket across forks. Close parent sockets before fork, reconnect inside the child.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [developerz-ai/wurk](https://github.com/developerz-ai/wurk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
