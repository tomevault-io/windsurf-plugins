---
trigger: always_on
description: This file is a working guide for future agents contributing to `xqueue`.
---

# AGENTS.md

This file is a working guide for future agents contributing to `xqueue`.

`SPEC.md` is the source of truth. This document compresses it into build-time
guidance so implementation decisions stay aligned.

## Mission

Build `xqueue` (`xq`), a minimal, single-machine, CLI-first, durable work queue
for running shell commands.

Primary use case:

1. `xcron` or another local process detects work
2. it enqueues command jobs into `xq`
3. `xq` workers run those jobs with explicit concurrency control

This project is intentionally local-first. It is not trying to become a
distributed queue, workflow engine, scheduler, or function-execution platform.

## Product Boundaries

`xqueue` is:

- a local durable queue for command execution
- a CLI-first operator tool
- a small control plane for one-machine worker management
- a complement to `xcron`

`xqueue` is not:

- a distributed broker
- a DAG or workflow engine
- a scheduler
- a Python callable runner
- a replacement for OS-level service supervision

## Core Model

The primary unit is a `job`.

A job is one command execution request plus execution metadata. The core model
is command-first, not callable-first.

Default operator UX:

```sh
xq enqueue --queue agent --cwd /repo -- "python scripts/check_mailbox.py --agent writer-1"
```

Important implications:

- v1 should treat shell mode as the primary execution mode
- storing one command string is sufficient for the primary UX
- direct exec/argv mode is optional later, not the default
- workers execute subprocesses and manage them via process groups
- process-group control is required for timeout and cancellation correctness

## Non-Negotiable Constraints

Future implementations should preserve these decisions unless `SPEC.md` changes:

- Python project managed with `uv` from day one
- Typer for CLI
- Rich only for `--output text`
- `--output json` must bypass presentation formatting entirely
- Pydantic models separate from SQLAlchemy ORM models
- YAML only for static configuration, never mutable queue state
- SQLite is the canonical state store
- Alembic is used from the beginning
- SQLite must run in WAL mode with explicit transactions and sensible
  `busy_timeout`
- stdout/stderr for job attempts live in files, not DB blobs
- application logs use `structlog`

## State and Reliability Model

Required durable job states in v1:

- `queued`
- `running`
- `succeeded`
- `failed`
- `retry_scheduled`
- `canceled`

Optional later states:

- `timed_out`
- `dead`

The system is at-least-once, not exactly-once. If a worker crashes after
claiming a job, that job may be retried after stale lease recovery.

Each attempt must be recorded separately. Top-level job state alone is not
enough.

## Required v1 Capabilities

- durable enqueue
- queue selection
- worker concurrency control
- atomic job claiming
- retries with delay
- per-job timeout
- cancellation of queued jobs
- cooperative stop for running jobs
- inspection via CLI
- machine-readable JSON output
- attempt history
- stdout/stderr capture
- stale lease recovery after worker crash

## Explicit v1 Non-Goals

- distributed workers
- remote broker mode
- DAG dependencies
- cron-like scheduling
- web UI
- plugin system
- exactly-once delivery

Do not smuggle these in through abstractions "for later" unless they have near
zero implementation cost and do not distort the local-first design.

## CLI Contract

The CLI is the main interface, not a thin wrapper.

Every important operation should be operable from `xq`. JSON output is a
first-class API for automation and agent use.

Conventions to preserve:

- default human-readable output
- `--output text|json` on operator-facing commands
- stable JSON field names and top-level shapes
- `list` commands return `{ "items": [...] }`
- `show` commands return `{ "item": { ... } }`
- successful mutations return `{ "ok": true, "item": { ... } }`
- failures should be structured where practical
- stable exit codes matter

Minimum command surface expected by the spec includes:

- `enqueue`
- `worker`
- `jobs list/show/cancel/retry/delete/tail/purge`
- `queues list/stats/pause/resume`
- `workers list/stop/pause/resume/drain`
- `config show`
- `controller run/install/uninstall/start/stop/restart/status`
- `health`
- `doctor`
- `recover stale-leases`
- `db check`
- `db vacuum`

## Worker and Lease Model

Workers are long-running local processes.

Responsibilities:

- atomically lease runnable jobs
- spawn subprocesses for commands
- heartbeat while jobs run
- capture stdout/stderr
- update job and attempt state
- honor timeout and cancellation
- requeue or fail according to policy

Worker operational states should be visible separately from job states:

- `active`
- `paused`
- `draining`
- `stopped`

Lease rules:

- claims must be transactional
- a claimed job is associated with one worker
- leases expire unless renewed
- workers heartbeat active leases
- stale leases enable crash recovery

## Cancellation and Timeout Semantics

For queued jobs:

- cancel moves directly to `canceled`

For running jobs:

1. record the request/reason
2. send `SIGTERM` to the process group
3. wait for configurable grace period
4. escalate to `SIGKILL` if needed
5. record the terminal outcome precisely


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cognesy/xqueue](https://github.com/cognesy/xqueue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
