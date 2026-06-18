---
trigger: always_on
description: Khora library: knowledge graphs + vector search + PostgreSQL for unified knowledge storage. **Library, not an application.**
---

# Khora

Khora library: knowledge graphs + vector search + PostgreSQL for unified knowledge storage. **Library, not an application.**

## Commands

```bash
make install           # uv sync --all-extras --no-extra google-adk (crewai combo)
make install-adk       # uv sync --all-extras --no-extra crewai     (google-adk combo)
make test              # pytest, coverage ≥65%
make format            # black, isort, ruff
make lint              # ruff + ty typecheck
make dev               # Start postgres + neo4j
uv run alembic upgrade head                       # Run migrations
```

**Installing all extras at once doesn't work.** `crewai` and `google-adk` declare mutually-incompatible `opentelemetry-api` ranges (crewai pins `<1.35`, google-adk pins `>=1.36`) and `[tool.uv].conflicts` makes the conflict explicit, so `uv sync --all-extras` is rejected. Pick one combo: `make install` for crewai (matches CI's `test` job), `make install-adk` for google-adk. CI also keeps `UV_NO_SYNC=1` set so subsequent `uv run` calls don't silently re-resolve the venv to the other combo.

CLI tooling (`extract`, `search`, ontology tools) was removed from the `khora` package. khora is a Python library.

## Test Commands

```bash
make test                                          # Full test suite (unit + integration + e2e), coverage ≥65%
uv run pytest -m integration                       # Integration tests only
uv run pytest -m e2e                               # End-to-end tests only
```

Docker Compose is always available. Always run `make test` before opening a PR. Never skip tests.

## Test Infrastructure Isolation

**Never reuse running Docker containers from other projects.** Integration tests must use their own Docker Compose stack (compose file in this repo), not containers from other worktrees or other developer projects. Before running integration tests:

1. Ensure your test databases are started from THIS repo's compose file
2. If port conflicts arise, stop your own containers or use different ports - never repurpose another project's infrastructure

## Architecture

- **Engines:** implement `MemoryEngineProtocol` in `engines/protocol.py`. Default engine is `vectorcypher`
- **Graph backends:** Neo4j, SurrealDB, Memgraph, Neptune, AGE - implement `GraphBackend` in `storage/backends/base.py`
- **SurrealDB:** unified backend (graph + vector + relational). Modes: `memory://`, `surrealkv://` (embedded), `ws://` (remote). Set `backend: surrealdb` in config
- **Extraction skills:** YAML-defined in `extraction/skills/builtin/`.
- **Config:** env vars with `KHORA_` prefix and single underscore (e.g., `KHORA_QUERY_ENABLE_HYDE=true`, `KHORA_LLM_MODEL=gpt-4o`). The legacy `__` nesting form is kept working silently for backward compatibility; single underscore is the documented form

### Key Entry Points

- `khora.py` - `remember()`, `recall()`, `forget()`, `remember_batch()`. Accepts `expertise: ExpertiseConfig`
- `extraction/skills/base.py` - `ExpertiseConfig`, `EntityTypeConfig`, `RelationshipTypeConfig`
- `storage/coordinator.py` - `transaction()` for atomic multi-backend ops
- `storage/backends/base.py` - `GraphBackend` protocol (implement for new backends)
- `storage/backends/surrealdb/` - Unified SurrealDB backend
- `db/models.py` - SQLAlchemy ORM (UUID columns use `as_uuid=True`)
- `_accel.py` - Rust/NumPy acceleration (MMR, cosine, pagerank, entity resolution, community detection, temporal)
- `extraction/binary_readers.py` - xlsx/docx/parquet readers; stable public boundary (`.pdf` raises `NotImplementedError` — preprocess upstream or use khora-cli).
- `pipelines/flows/ingest.py` - Document ingestion pipeline (3-phase: stage → enrich → expand)
- `db/migrations/env.py` - Alembic with advisory locking
- `config/schema.py` - `KhoraConfig` Pydantic settings (storage, LLM, pipeline, query, tenancy)
- `exceptions.py` - `KhoraError` hierarchy with domain-specific exceptions
- `telemetry/` - Optional PostgreSQL-backed telemetry collector + `@trace` decorator

## Issue tracking & workflow

khora is open source. **All khora work is tracked in GitHub Issues** at https://github.com/DeytaHQ/khora/issues. Use `gh issue` from the CLI or the GitHub web UI.

Workflow for any change:

1. Create or pick a GitHub issue describing the work.
2. Create a feature branch off `main` (`<initials>/<short-desc>`).
3. Open a PR against `main`. Include `Fixes #<n>` in the body to auto-close the issue on merge.
4. CI must be green before merge. Squash-merge by default.
5. The `release.yml` workflow publishes to PyPI on `v*` tag push (see `docs/RELEASE.md`).

**Do not maintain `docs/AI_CHANGELOG.md` in this repo.** Commit messages and merged-PR titles are the changelog of record.

## Conventions

### Version Bumps

Khora uses `hatch-vcs` - khora's version comes from git tags (`git tag vX.Y.Z`). khora-accel has its version in source. **khora and khora-accel are always released at the same version (lockstep contract)** - the matching pin in `pyproject.toml`'s `rust` extra enforces this for installers.

Per release:

1. `rust/khora-accel/Cargo.toml` - update `version = "X.Y.Z"`
2. `pyproject.toml` (root) - update `khora-accel == X.Y.Z` in the `rust` extra to match

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DeytaHQ/khora](https://github.com/DeytaHQ/khora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
