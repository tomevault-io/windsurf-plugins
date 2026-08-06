---
trigger: always_on
description: Quick orientation file for AI coding assistants and any human cracking
---

# bqemulator — orientation for AI assistants and new contributors

Quick orientation file for AI coding assistants and any human cracking
the repo open for the first time. Pairs with
[docs/architecture/overview.md](docs/architecture/overview.md), the
canonical architectural reference.

## What this project is

Open-source local emulator for Google BigQuery. Python 3.11+, DuckDB-backed,
SQLGlot-powered. Drop-in replacement for the real service for dev, CI, and
offline replicas. Apache 2.0.

## Non-negotiable principles

1. **Highest engineering standards.** Hexagonal architecture, dependency
   injection, strict types, pattern-driven (Strategy, Command, Repository,
   Visitor, Interpreter). No shortcuts.
2. **≥90% line + branch coverage.** CI fails below threshold. No exceptions.
3. **E2E against live Docker containers.** Every scenario passes against
   `ghcr.io/jjviscomi/bqemulator:dev` for all five conformance clients
   (Python, Node.js, Go, Java SDKs + Google's `bq` CLI) before merge.
4. **Comprehensive docs with runnable examples.** Every user-facing feature
   has a guide AND a runnable CI-verified example. `mkdocs build --strict`
   in CI.
5. **No deferral.** When starting a feature, complete it. Scope boundaries
   are explicit exclusions with documented rationale — never "TODO for
   v1.1". See [`docs/reference/out-of-scope.md`](docs/reference/out-of-scope.md).

## Quick commands

| Command | What it does |
|---|---|
| `make dev-setup` | Install deps + pre-commit hooks |
| `make verify` | Full release-ready gate chain (lint → unit → property → integration → docker → e2e → docs) |
| `make lint` | ruff + format + mypy --strict + bandit + pip-audit + interrogate + typos |
| `make test-unit` | Unit tests, <10s |
| `make test-property` | Hypothesis property tests |
| `make test-integration` | In-process emulator + Python client |
| `make docker-build` | Build multi-arch image `ghcr.io/jjviscomi/bqemulator:dev` |
| `make test-e2e` | Live container + all five conformance clients |
| `make test-conformance` | Replay corpus against in-process emulator (offline) |
| `make record-conformance` | Re-record corpus baselines from real BigQuery (requires `GOOGLE_APPLICATION_CREDENTIALS` + `BQEMU_CONFORMANCE_PROJECT`) |
| `make coverage-matrix` | Regenerate `docs/reference/conformance-coverage-matrix.md` |
| `make test-perf` | pytest-benchmark, regressions >10% fail |
| `make docs-serve` | Local MkDocs preview |
| `make docs-build` | `mkdocs build --strict` |
| `make release-dry-run NEXT=minor` | Preview a release |
| `make release NEXT=minor` | Apply a release (verify + bump + changelog + commit + tag) |
| `bqemulator start --ephemeral` | Start emulator for manual debugging |
| `bqemulator start --data-dir /tmp/bqemu` | Persistent mode |

## Architecture (hexagonal)

```
api/  ──┐
        ├── domain/ + catalog/ + storage/ + sql/ + jobs/ + streaming/
grpc_api/ ──┘        + scripting/ + udf/ + versioning/ + types/
                     + row_access/ + views/ + commands/
```

- `src/bqemulator/domain/` — framework-free pure domain (errors, `Result`,
  clock, IDs, events).
- `src/bqemulator/catalog/` — metadata, Repository pattern, in-memory +
  DuckDB-backed implementations.
- `src/bqemulator/storage/` — DuckDB engine + type mapping + Arrow bridge
  + partition state.
- `src/bqemulator/sql/` — SQLGlot orchestrator + rule strategies +
  rewriters + query cache + built-in UDFs.
- `src/bqemulator/scripting/` — BigQuery scripting interpreter
  (`DECLARE` / `BEGIN` / `END` / `IF` / `LOOP` / `EXCEPTION` +
  `BEGIN`/`COMMIT`/`ROLLBACK` transaction shim per
  [ADR 0015](docs/adr/0015-scripting-execution-model.md)).
- `src/bqemulator/udf/` — SQL / JS (V8) / TVF runtimes.
- `src/bqemulator/versioning/` — snapshots, time-travel, clones,
  materialized views.
- `src/bqemulator/jobs/` — command-pattern executor for query / load /
  extract / copy / snapshot.
- `src/bqemulator/streaming/` — Storage Read (Arrow + Avro) + Storage
  Write APIs (strategy per stream type) + proto / Arrow row
  deserialisers.
- `src/bqemulator/row_access/` — RAP enforcement via rewriter.
- `src/bqemulator/views/` — authorized views.
- `src/bqemulator/types/` — `GEOGRAPHY`, `RANGE`, `INTERVAL`, numeric,
  timestamp.
- `src/bqemulator/commands/` — CLI subcommands (`start`, `import`,
  `admin`, …) routed by `cli.py`.
- `src/bqemulator/api/` — FastAPI REST adapter (incl. multipart +
  resumable upload routes per
  [ADR 0029](docs/adr/0029-upload-host-endpoints.md)).
- `src/bqemulator/grpc_api/` — `grpc.aio` adapter (Storage Read + Storage
  Write servicers).
- `src/bqemulator/observability/` — `structlog`, OpenTelemetry, Prometheus.
- `src/bqemulator/testing/` — testcontainers helpers + pytest plugin
  entry points.

## Conventions

- **GitHub Actions pinning.** **Every** `uses:` reference in
  `.github/workflows/*.yml` is pinned to a **full-length commit SHA**
  with a trailing `# vX.Y.Z` comment that names the matching release
  tag — including first-party `actions/*`. SHA pinning is the
  [GitHub Security Lab](https://github.blog/security/supply-chain-security/four-tips-to-keep-your-github-actions-workflows-secure/)
  + [OpenSSF Scorecard](https://github.com/ossf/scorecard/blob/main/docs/checks.md#pinned-dependencies)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jjviscomi/bqemulator](https://github.com/jjviscomi/bqemulator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
