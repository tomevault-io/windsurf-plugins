---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

**Placeframe** is a self-hosted XR spatial localization system ("relocalization as a service"). It determines an XR device's position and rotation relative to a canonical reference frame for a physical space — an open-source alternative to Apple Shared World Anchors, Google ARCore Cloud Anchors, etc.

The repo also hosts two Unity apps that consume Placeframe: `apps/AndroidMobile/` (the Capture Tool — pending rename) and `apps/MakeItSing/` (a multiplayer XR client). Each app directory has its own `CLAUDE.md` and `SPEC.md` for app-specific guidance.

## Documentation

The repo uses a two-tier docs model: `CLAUDE.md` for always-loaded rules (prescriptive, terse, under ~80 lines per file) and `SPEC.md` for on-demand narrative co-located with the code it describes. The authoring rules for `SPEC.md` are locked in `/SPEC-STYLE-GUIDE.md` — **read it before authoring or editing any `SPEC.md`**.

- **Prose and code commit separately.** Markdown (`*.md`) and source code never share a commit, even when changed in the same session. This keeps prose diffs reviewable on their own terms.
- **Spec-first on disagreement.** When a `SPEC.md` and the code it describes disagree, update the spec first and surface the diff, *then* change the code. This converts every disagreement from silent rot into an explicit human decision.
- **No `docs/` directory.** Cross-cutting content lives in a parent-directory `SPEC.md` or a top-level special file (`README.md`, `SPEC-STYLE-GUIDE.md`, etc.).

## Commands

All top-level commands are run via `uv run <command>` from the repo root. These are defined in `scripts/src/scripts/` and registered in `scripts/pyproject.toml`.

| Command | Purpose |
|---|---|
| `uv run up` | Start all Docker services (detached). Pass `--attached` for streaming logs. |
| `uv run down` | Stop all Docker services |
| `uv run build` | Build Docker images (auto-detects CUDA/ROCm) |
| `uv run migrate-database` | Run PostgreSQL schema migrations |
| `uv run generate-clients` | Regenerate OpenAPI client packages |
| `uv run generate-datamodels` | Regenerate Pydantic data models |
| `uv run lock-python` | Regenerate workspace `uv.lock` and per-service `pylock.toml` files |
| `uv run deptry-check` | Check for dependency issues across all packages |

**Linting and type checking** (run from repo root):
```bash
uv run ruff check .          # Lint
uv run ruff format .         # Format
uv run basedpyright          # Type check (strict mode)
```

**Tests**: `uv run pytest` from repo root. Tests live alongside each service (e.g. `docker/localizer/tests/`).

**Full preflight**: `uv run --no-sync preflight` from the repo root. This is the exact command CI invokes — it bundles sync, lint, format check, type check, deptry, pytest, lock-file check, datamodel codegen, and client codegen staleness, all gated as a single pass/fail. Run it before claiming a change is CI-clean; running individual checks (just `ruff check`, just `pytest`, just the codegen check) won't catch failures in the others. Note: preflight tears down and re-brings-up `compose.postgres.yml`, so it interrupts a running stack.

## Server stack

The server stack (API, localizer, reconstructor, Keycloak, MinIO, Postgres, Loki/Alloy/Grafana) is a set of Docker microservices under `docker/`. The reconstructor pulls work via API lease endpoints — there is no separate orchestrator service. See `docker/SPEC.md` for service inventory, data flow, authentication model, and operational debugging (Postgres / MinIO / Loki access patterns, reconstructor lease lifecycle).

## Python Workspace

The repo is a `uv` monorepo. Shared Python code lives in `packages/python/`:

- **`common`** — utilities for boto/MinIO, Docker SDK, Litestar, JWT
- **`core`** — domain logic: camera configs, coordinate transforms, metrics
- **`neural-networks`** — PyTorch models with conditional extras (`cpu`, `cuda`, `rocm`)
- **`datamodels`** — auto-generated Pydantic models from the OpenAPI schema
- **`api-client` / `localizer-client`** — auto-generated async API clients

Auto-generated packages in `packages/generated/` should not be edited directly — regenerate them with the commands above.

**Generation pipeline**: Code in `packages/generated/` is produced by two scripts that must be run after certain changes:

- **`uv run generate-datamodels`** — Introspects the **live PostgreSQL database** (via `sqlacodegen`) to produce `packages/generated/python/datamodels/` (SQLAlchemy table models + Pydantic DTOs). Must be run after any changes to `database/*.sql` schema files. **Requires Docker + postgres to be running** (`uv run up`, then `uv run migrate-database` to apply schema changes).
- **`uv run lock-python`** — Regenerates the workspace `uv.lock` and per-service `pylock.toml` files (for services with a `Dockerfile`). Must be run before `generate-clients` (which uses `uv run --no_workspace` per-service and needs the lock files). Also re-run after `uv sync --all-packages` since that overwrites per-service locks.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [outernet-foundation/placeframe](https://github.com/outernet-foundation/placeframe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
