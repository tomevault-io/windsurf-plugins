---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) and other ZCode agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) and other ZCode agents when working with code in this repository.

## Project Overview

Mini-HF is a LAN-focused model cache repository system for HuggingFace/ModelScope. It provides HF Hub-compatible APIs to accelerate model downloads within a local network while reducing external bandwidth usage. Three FastAPI servers + a worker, backed by PostgreSQL + Redis + S3-compatible storage. React 19 SPA frontend.

## Architecture

### Backend Package Structure

All backend code is in `packages/` as a uv workspace:

| Package | Path | Purpose |
|---------|------|---------|
| `core` | `packages/core` | Configuration management (`core.settings`) |
| `database` | `packages/database` | SQLAlchemy async models, repositories |
| `cache` | `packages/cache` | Redis cache client and progress tracking |
| `storage` | `packages/storage` | S3-compatible client (boto3) |
| `services` | `packages/services` | HuggingFace/ModelScope service clients |
| `mgmt_server` | `packages/mgmt_server` | Management API (Port 9800) |
| `hf_server` | `packages/hf_server` | HF-compatible API (Port 9801) |
| `ms_server` | `packages/ms_server` | ModelScope-compatible API (Port 9802) |
| `worker` | `packages/worker` | Task processor |

**Dependency chain**: `mgmt_server` / `hf_server` / `ms_server` / `worker` → `database` / `cache` / `storage` / `services` → `core` (settings). Each server and the worker depend on the infrastructure packages, which all depend on `core` for configuration. Don't introduce cycles.

### Settings (`packages/core/src/core/settings.py`)

All configuration is defined via pydantic-settings in a single `Settings` class. Key worker tuning knobs live here:
- `WORKER_POLL_INTERVAL`, `WORKER_MAX_CONCURRENT`, `WORKER_CANCEL_CHECK_INTERVAL`
- `WORKER_CONCURRENT_DOWNLOADS` / `WORKER_CONCURRENT_UPLOADS` / `WORKER_CONCURRENT_S3_CHECKS`
- `WORKER_PROGRESS_INTERVAL`, `WORKER_MAX_RETRIES`, `WORKER_RETRY_BASE_DELAY`, `WORKER_RETRY_MAX_DELAY`

Import via `from core.settings import settings` (module-level singleton).

### Database Layer (`packages/database`)

**Models**: `packages/database/src/database/db_models/` — SQLAlchemy async ORM models. Key entities: `HfRepoProfile`, `HfRepoSnapshot`, `HfRepoTreeItem`, `Task`, `User`, `Announcement`, `SystemConfig`.

**Repositories**: `packages/database/src/database/db_repositories/` — Data access classes that encapsulate SQL queries. Each entity has a dedicated repository (e.g., `TaskRepository`, `HfRepoProfileRepository`).

**Session management** (`packages/database/src/database/core.py`):

- `unit_of_work()` — **Preferred** FastAPI dependency. Commits on success, rolls back on exception, always closes. Use via `Depends(unit_of_work)`.
- `new_session()` — Creates a session; caller manages commit/rollback/close manually. Use in non-FastAPI contexts (worker, scripts).
- `get_db()` / `get_session()` — **Deprecated** aliases. Do not use in new code.

**Alembic**: `alembic.ini` at repo root, migrations in `alembic/versions/`. The `env.py` constructs the DB URL from `PG_*` environment variables (not from settings.py), so migrations need those env vars set.

### Worker Architecture (`packages/worker`)

The worker runs a polling loop that picks up `PENDING` tasks and processes them through a **6-phase download workflow** defined in `BaseDownloadHandler` (`packages/worker/src/worker/handlers/base_handler.py`):

1. `prepare_profile` — Set repo profile status to UPDATING
2. `resolve_commit` — Resolve source endpoint and commit hash
3. `calculate_diff` — Compare new tree against old snapshot, compute file diff (download/update/delete)
4. `save_tree` — Persist snapshot and tree items to database
5. `execute_downloads` — Download from source, upload to S3 (with concurrency semaphores)
6. `finalize_success` — Activate snapshot, set profile ACTIVE, cleanup

The handler is split into four protocol ABCs (Interface Segregation): `ProfileLifecycle`, `TreeLifecycle`, `DownloadInfrastructure`, `CleanupLifecycle`. Source-specific subclasses (e.g., `HfDownloadHandler` in `handlers/hf/handler.py`) implement these protocols.

Key worker modules:
- `handlers/base_handler.py` — Template method orchestrating the 6 phases
- `handlers/diff_calculator.py` — Compares old vs new file trees
- `handlers/file_processor.py` — Concurrent download+upload pipeline
- `handlers/downloader.py` — Byte-transfer implementation (resume, retry)
- `handlers/download_context.py` — Shared state object passed through phases
- `handlers/progress_tracker.py` — Redis-backed progress tracking
- `handlers/contracts.py` — `TaskControl` (cancel/pause signals) and `ExecutionResult`
- `handlers/source_types.py` — Source endpoint type definitions

The worker loop itself (not a handler) lives at the package root of `worker`:
- `worker.py` — `Worker` class, the polling loop that picks up `PENDING` tasks
- `watchdog.py` — `TaskWatchdog`, a background coroutine that batch-checks running tasks for `CANCELING`/`PAUSING` DB transitions (replaces per-task watchers)
- `retry.py` — `RetryPolicy`, decides whether/how to retry a failed task (backoff in `settings.WORKER_RETRY_*`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [realtyz/mini-hf](https://github.com/realtyz/mini-hf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
