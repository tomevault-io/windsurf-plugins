---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository status

**1.0 shipped; 1.x cloud-Redis polish landed.** Phases 1–4 shipped (engine, delayed jobs + retries, Node bindings, Python bindings + CLI). 1.0 polish closed the three PRD-listed release blockers:

- Function-reference enqueue: stable `jobId` + `Queue.addUnique` on both shims.
- Per-job result backends: opt-in `Queue.getJobResult` + `Job.waitForResult` polling helpers.
- Bench coverage with a non-Rust handler: `python_handler_bench.py` + the Criterion FFI buffer-copy microbench.

Slice 11 (May 2026) added cloud-Redis prerequisites — TLS via `rediss://`, `ConnectionTuning` for TCP keepalive + reconnect policy, `Producer::shutdown` clean disconnect, and a `CredentialProvider` hook for rotating-token auth (ElastiCache IAM). PRs #114-#118 in `docs/history.md`. v1.3.0 (May 2026) extended the credential hook across FFI: the Node shim takes `connection.credentialProvider` and the Python shim takes `credential_provider` (PRs #132–#133).

**Slice-by-slice engineering history lives in [`docs/history.md`](docs/history.md)** — read that for the long-form context (Phase 2 slices, name-on-wire, the post-#62 polish slices, the slice-11 cloud-Redis work, etc.). This file is the orientation doc; treat the history file as the changelog.

**Deferred 1.x follow-ups:**

- Opt-in result-write bench scenario (`store_results=true` sustained throughput).
- `maxmemory` eviction-behavior verification (engine behavior under Redis eviction policies has not been exercised end-to-end).
- All-primaries `SCRIPT LOAD` preload (cluster startup optimization — the `NOSCRIPT`→`EVAL` self-heal already makes cluster correct; eager preload is an optional perf nicety, see `docs/history.md`).

Cross-FFI credential-provider callbacks for the Node and Python shims shipped in v1.3.0 (PRs #132–#133) — no longer deferred.

Redis Cluster support shipped (May 2026) — connect with a `redis-cluster://` / `rediss-cluster://` URL (or Node `connection.cluster: true`). The engine was already cluster-correct (the `{chasqui:<queue>}` hash tag co-locates a queue's keyspace on one slot; every command uses `ClusterHash::FirstKey`); the slice fixed two shim TLS-URL bugs, added a real-cluster integration test + CI job, and synced docs. See `docs/history.md`.

Stalled-job detector shipped (May 2026, v1.4.0) — leader-elected background task spawned alongside the promoter/scheduler that bounds worker-crash loops independently of handler-failure loops. New `DlqReason::Stalled`, `MetricsSink::stalled_tick`, `e=stalled` event, `JobInfo.stalled_count`, and a Node-shim **BREAKING CHANGE**: `WorkerOptions.maxStalledCount` now routes to engine `max_stalled_attempts` (the correct semantic — stall cycles before DLQ-as-`stalled`) instead of `max_attempts` (total handler attempts) — with a one-time warn-once when users hit the migration cell. Python shim adds `Worker(max_stalled_attempts=...)` clean (no deprecation needed — Python never had the mis-routed field). See `docs/history.md`.

## Workspace

- `chasquimq/` — engine crate.
- `chasquimq-node/` — Node.js bindings via `napi-rs` + high-level shim.
- `chasquimq-py/` — Python bindings via PyO3 + high-level shim.
- `chasquimq-cli/` — `chasqui` operator binary.
- `chasquimq-bench/` — bench harness (parallel to `bullmq-bench`).
- `chasquimq-metrics/` — opt-in `MetricsSink` → `metrics-rs` / Prometheus adapter.

CI: `.github/workflows/ci.yml` (rustfmt, clippy `--all-targets --workspace -- -D warnings`, `cargo test --workspace -- --include-ignored` against a `redis:8.6.2` service container) — runs on push to `main` and every PR. Plus `.github/workflows/{node-ci,py-ci,cross-shim,release}.yml`.

## Key files for context

- `README.md` — public-facing pitch, headline numbers, three quickstarts, feature comparison.
- `CONTRIBUTING.md` — dev setup, PR workflow, commit conventions, in/out of scope.
- `docs/engine.md` — engine internals: retry semantics, delayed jobs, DLQ tooling, result backends, observability sinks, operational notes.
- `docs/history.md` — slice-by-slice engineering history.
- `prd/prd.md` — product requirements, source of truth for product intent.
- `benchmarks/README.md` — index for all bench reports.
- `benchmarks/baseline-bullmq.md` — measured BullMQ baseline on this host. **The numbers ChasquiMQ has to beat live here.** Read it before making any perf-related design choice.
- `benchmarks/chasquimq-1.0.md` — same-host 1.0 re-bench (today's contended-host BullMQ + ChasquiMQ side-by-side).
- `benchmarks/runs/` (gitignored) — raw logs land here locally; only the summary `.md` files are committed.

When updating user-facing docs, keep `README.md`, `CONTRIBUTING.md`, `benchmarks/README.md`, and this file in sync. Don't duplicate content across them — link instead.

The upstream BullMQ benchmark suite is **not vendored** — it's cloned at `~/Projects/experiments/bullmq-bench` (sibling to this repo). Treat it as external; don't edit it.

## Product


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jotarios/chasquimq](https://github.com/jotarios/chasquimq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
