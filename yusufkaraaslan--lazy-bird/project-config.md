---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Lazy-Bird is a development automation system that lets Claude Code instances work on software development tasks autonomously. It monitors GitHub/GitLab issues labeled `ready`, creates isolated git worktrees, runs Claude Code in containers, executes tests, and creates PRs. Supports 18+ frameworks (Godot, Django, React, Rust, etc.).

**Version:** 2.0.0-alpha (FastAPI microservice rewrite of the original Flask/bash v1.x)
**Python:** >= 3.10
**Package:** `pip install lazy-bird` (entry point: `lazy-bird` CLI)
**Install dev deps:** `pip install -e ".[dev]"`

## Commands

```bash
# Testing
pytest                                    # Run all tests
pytest tests/unit/                        # Unit tests only
pytest tests/integration/                 # Integration tests only
pytest tests/unit/test_claude_service.py  # Single test file
pytest -k "test_health"                   # Tests matching pattern
pytest --cov=lazy_bird --cov-report=html  # With coverage report

# Code quality
black lazy_bird/ tests/                   # Format (line-length=100)
flake8 lazy_bird/ tests/                  # Lint
mypy lazy_bird/                           # Type check (strict mode)
bandit -r lazy_bird/                      # Security scan

# Run the API server directly
python -m lazy_bird.api.main              # Starts uvicorn on :8000

# Celery workers (requires Redis)
celery -A lazy_bird.tasks worker --loglevel=info
celery -A lazy_bird.tasks beat --loglevel=info

# Legacy CLI (delegates to bash scripts)
lazy-bird setup                           # Setup wizard
lazy-bird server                          # Start old Flask backend on :5000
lazy-bird project list                    # List projects
```

## Architecture

### v2.0 Core (FastAPI + SQLAlchemy + Celery)

The v2.0 backend is a FastAPI application with async PostgreSQL and Celery task processing:

```
lazy_bird/
  api/
    main.py              # FastAPI app, lifespan, router registration
    dependencies.py      # DI: get_async_database, get_current_api_key, RequireScopes
    middleware.py         # CORS, logging, error handling middleware
    exceptions.py        # Exception handlers registered on the app
    routers/             # One router per resource, all mounted at /api/v1
      health.py, projects.py, task_runs.py, claude_accounts.py,
      framework_presets.py, api_keys.py, webhooks.py, auth.py
  core/
    config.py            # Pydantic Settings (loads from .env), cached via get_settings()
    database.py          # SQLAlchemy engines (sync + async), session factories, Base
    security.py          # JWT creation/verification, API key hashing
    redis.py             # Redis connection management
    logging.py           # Structured logging setup
  models/                # SQLAlchemy ORM models (all inherit from Base)
    project.py, task_run.py, task_run_log.py, claude_account.py,
    framework_preset.py, webhook_subscription.py, daily_usage.py, api_key.py, user.py
  schemas/               # Pydantic request/response schemas
    project.py, task_run.py, framework_preset.py, claude_account.py,
    api_key.py, webhook.py, user.py
  services/              # Business logic layer
    claude_service.py    # Executes Claude Code CLI, parses output, tracks tokens
    git_service.py       # Git worktree creation/cleanup, branch management
    test_runner.py       # Framework-specific test execution
    pr_service.py        # GitHub/GitLab PR creation
    webhook_service.py   # Webhook delivery with retries
    log_publisher.py     # Real-time log streaming via Redis Pub/Sub
    preset_seeder.py     # Seeds framework presets from YAML on startup
  tasks/
    __init__.py          # Celery app instance
    celeryconfig.py      # Celery configuration
    task_executor.py     # Main Celery task: execute_task(task_run_id)
    queue_processor.py   # Polls for queued TaskRuns
    issue_watcher.py     # Watches GitHub/GitLab for 'ready' issues
  cli.py                 # CLI entry point (argparse, delegates to scripts/)
```

### Key Patterns

- **Config:** `lazy_bird.core.config.settings` is a cached `pydantic_settings.BaseSettings` singleton loaded from `.env`. Import it directly or use `get_settings()`.
- **Database:** Dual engine setup - async (asyncpg) for FastAPI endpoints, sync for Celery/migrations. All models inherit from `lazy_bird.core.database.Base`. Tests use in-memory SQLite via `aiosqlite`.
- **Dependencies:** FastAPI DI via `dependencies.py`. Auth is `X-API-Key` header or `Authorization: Bearer <jwt>`. Scope-based permissions via `RequireScopes(["write", "admin"])`. Convenience aliases: `RequireRead`, `RequireWrite`, `RequireAdmin`.
- **Task execution flow:** API creates TaskRun (status=queued) -> Celery picks it up -> creates git worktree -> runs Claude Code CLI (`claude -p "prompt"`) -> runs tests -> creates PR -> updates status. Webhooks fire on state changes.
- **Migrations:** Alembic is configured (`alembic/`) but no migrations exist yet. Schema is created via `Base.metadata.create_all()` in dev/test.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yusufkaraaslan/lazy-bird](https://github.com/yusufkaraaslan/lazy-bird) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
