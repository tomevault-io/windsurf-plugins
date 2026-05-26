---
trigger: always_on
description: Forge is a Rust control plane for deterministic deployment and runtime convergence of application containers on a single node. It is not a generic PaaS and it is not a multi-node scheduler. The core product contract is:
---

# AGENTS.md

## Project Overview

Forge is a Rust control plane for deterministic deployment and runtime convergence of application containers on a single node. It is not a generic PaaS and it is not a multi-node scheduler. The core product contract is:

`running container != successful deployment`

A deployment is only successful after the full lifecycle completes:

`candidate -> validated -> finalized -> activated -> promoted`

The repository contains one Rust crate (`forge_core`) that builds the `forge` binary. That binary currently serves both operator CLI and daemon/server responsibilities. Product docs describe a future `forged` server binary, but the current codebase still ships a single binary.

## Architecture Summary

- Control-plane authority lives in the daemon and convergence loop, not in HTTP handlers, CLI parsing, Docker adapters, or Caddy adapters.
- The daemon is restart-safe and persists operational truth under `storage_root` using filesystem-backed state and atomic writes.
- Readiness is cache-backed: convergence computes truth asynchronously, `/readyz` and `/metrics` serve cached truth in constant time.
- Docker is the runtime substrate. Caddy is the HTTP routing substrate. Git/GitHub integration exists for source resolution and web login/webhooks.
- The system is explicitly single-writer. Lease-based leadership and split-brain detection scaffolding exist, but this is not consensus and not true HA.

Primary high-risk domains:

- `src/deployments.rs`: deployment FSM, validation, route activation, runtime env snapshotting
- `src/convergence.rs`: steady-state repair, rollback, retention, runtime truth
- `src/daemon.rs`: readiness cache, leadership, replay, convergence refresh, control-plane metrics
- `src/storage.rs`: atomic persistence, locks, pointer semantics, checkpoint/snapshot/journal files
- `src/reconciliation.rs`: intent log and replay safety
- `src/http.rs` and `src/auth.rs`: auth, OAuth, token issuance/verification, idempotency, API surface
- `src/upgrade.rs`, `scripts/package-release.sh`, `scripts/publish-release.sh`, `install.sh`: upgrade and release integrity

## Repository Structure

```text
src/
  main.rs              CLI entrypoint and daemon wiring
  lib.rs               module exports
  http.rs              Axum router, API handlers, web login/app serving
  daemon.rs            daemon lifecycle, readiness cache, leadership, metrics
  deployments.rs       deploy execution pipeline and promotion gates
  convergence.rs       steady-state reconciliation, rollback, GC/retention
  storage.rs           filesystem-backed stores, pointers, checkpoints, journals
  runtime.rs           Docker/routing traits and runtime contracts
  docker.rs            Docker CLI adapter
  caddy.rs             Caddy admin API adapter
  config.rs            `forge.conf` parsing
  secrets.rs           secret encryption and storage
  projects.rs          project registry and domain derivation
  backups.rs           persistent-volume backup and restore flow
  status.rs            status/diagnostics/history/env reporting
  upgrade.rs           signed release upgrade plan/apply/rollback
  ...
tests/
  cli.rs               CLI contract tests
  e2e.rs               dogfood end-to-end control-plane tests
  docker_integration.rs
  caddy_integration.rs
  release.rs           packaging/install/publish/upgrade tests
  integration/common.rs
  fixtures/            sample deploy targets
docs/
  ARCHITECTURE.md
  INVARIANTS.md
  OPERATIONS.md
  USAGE.md
  LOCAL_QUICKSTART.md
deploy/
  forge.conf.example
  forge.service
scripts/
  package-release.sh
  publish-release.sh
schemas/
  JSON schemas for project, deployment request, events, snapshots
web/
  static HTML/CSS/JS embedded by `src/http.rs`
openapi.yaml           HTTP API contract
dist/                  generated release artifacts
target/                Cargo build output
```

## Core Technologies

- Rust edition `2024`
- Axum for HTTP server
- Tokio runtime
- Reqwest with `rustls`
- Docker CLI integration for runtime control
- Caddy admin API integration for routing
- Serde/JSON/YAML
- AES-GCM for secret storage
- Static web assets embedded with `include_str!`

There is no monorepo/workspace structure in this repository. There is no frontend build toolchain visible; `web/` is served as static embedded assets.

## Repository-Specific Invariants

Read `docs/INVARIANTS.md` before changing deploy, convergence, rollback, pointer, snapshot, replay, or routing code.

Non-negotiable rules enforced by docs, code structure, and tests:

- Never advance `current` before validation and route activation succeed.
- `previous` must always point to the most recent superseded healthy generation only.
- Failed generations must never become rollback targets.
- `snapshot.json` is immutable once finalized.
- Queue state is not deployment truth.
- Rollback restores runtime topology, not database history.
- Restore creates a new generation and must not mutate persistent volumes in place.
- `/readyz` must stay constant-time and must not trigger live Docker/Caddy scans or fleet-wide recomputation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anggaprytn/forge-core](https://github.com/anggaprytn/forge-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
