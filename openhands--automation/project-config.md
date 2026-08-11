---
trigger: always_on
description: Self-contained microservice that schedules and dispatches automation runs inside OpenHands Cloud sandboxes.
---

# Automations Service

Self-contained microservice that schedules and dispatches automation runs inside OpenHands Cloud sandboxes.

## Repository Structure

```
automation/
├── openhands/
│   └── automation/          # Main application package (openhands.automation namespace)
│       ├── app.py              # FastAPI app, lifespan, background tasks
│       ├── auth.py             # Auth via OpenHands /api/v1/users/me (API key + cookie)
│       ├── config.py           # Pydantic settings (Settings, env prefix AUTOMATION_)
│       ├── constants.py        # Timeouts, polling intervals, sandbox constants
│       ├── db.py               # Database engine and session factory (asyncpg / Cloud SQL)
│       ├── dispatcher.py       # Polls PENDING runs, dispatches to sandbox (fire-and-forget)
│       ├── execution.py        # Sandbox lifecycle: create → upload → execute → delete
│       ├── logger.py           # JSON structured logging configuration
│       ├── models.py           # SQLAlchemy models (Automation, AutomationRun, TarballUpload)
│       ├── router.py           # API routes (CRUD, trigger, callback, runs list)
│       ├── scheduler.py        # Cron scheduler — polls automations, creates PENDING runs
│       ├── schemas.py          # Pydantic request/response schemas
│       ├── uploads.py          # Tarball upload router
│       ├── watchdog.py         # Staleness watchdog — marks hung runs as FAILED
│       ├── storage/            # File storage abstraction
│       │   ├── file_store.py   # Abstract base class for file storage
│       │   └── google_cloud.py # GCS implementation
│       └── utils/              # Utility modules
│           ├── api_key.py      # Per-user API key minting via service key
│           ├── cron.py         # Cron schedule utilities (next/prev fire time)
│           ├── run.py          # Run status transitions (create, mark, update)
│           ├── sandbox.py      # Sandbox verification and cleanup
│           ├── tarball_validation.py  # Tarball path validation (internal/external)
│           └── time.py         # UTC time helpers
├── containers/
│   └── Dockerfile          # Container image definition
├── migrations/              # Alembic migrations
├── scripts/
│   ├── test_automation.py  # E2E test (sandbox lifecycle with live streaming)
│   └── test_tarball/       # Tarball contents uploaded to sandbox during test
│       ├── main.py         # Test script run inside sandbox (SDK workspace test)
│       └── setup.sh        # Installs SDK inside sandbox
├── tests/                   # Unit tests (flat structure, no external deps)
│   ├── integration/        # Integration tests (require OPENHANDS_API_KEY)
│   ├── test_auth.py
│   ├── test_dispatcher.py
│   ├── test_execution.py
│   ├── test_router.py
│   ├── test_scheduler.py
│   └── ...
└── pyproject.toml
```

## Cross-Repo Coordination

Three repos work together:

| Repo | Branch | Purpose |
|------|--------|---------|
| `OpenHands/automation` | `dispatch-phase1b` | Automation service (this repo) |
| `OpenHands/deploy` (aka `All-Hands-AI/deploy`) | `dispatch-phase1b` | Deploys automation as a sidecar |
| `OpenHands/software-agent-sdk` | `feat/saas-runtime-mode` | SDK changes for in-sandbox execution |

**AUTOMATION_SHA linking**: The deploy repo references a specific automation commit in two workflow files:
- `.github/workflows/deploy.yaml` → `AUTOMATION_SHA: "<full-sha>"`
- `.github/workflows/deploy-automation.yaml` → `AUTOMATION_SHA: "<full-sha>"`

After pushing to the automation repo, update both files in the deploy repo.

## Configuration

Configuration is centralized in `config.py` using a composed `AppConfig` with typed sections:

```python
from automation.config import get_config

config = get_config()
config.service.db_host          # ServiceSettings (AUTOMATION_ prefix)
config.storage.file_store       # StorageSettings (no prefix, SDK conventions)
config.http.auth_cache_ttl      # HttpSettings (AUTOMATION_ prefix)
config.sandbox.max_run_duration # SandboxSettings (AUTOMATION_ prefix)
config.kv.kv_secret             # KVSettings (AUTOMATION_ prefix)
config.log.log_level            # LogSettings (no prefix)
```

**Key principles:**
- Use `get_config().<section>` instead of deprecated `get_settings()`
- All environment variables documented in config class docstrings
- Protocol constants (WORK_DIR, TARBALL_PATH) in `constants.py` - these cannot be changed without breaking compatibility
- Shared logging context via `log_extra()` from `automation.utils`

## Build & Test Commands

```bash
# Pre-commit (run from repo root)
pre-commit run --files openhands/**/*.py scripts/**/*.py tests/**/*.py --show-diff-on-failure

# Unit tests (no external deps, skips Docker-dependent tests)
uv run pytest tests/ -v --ignore=tests/integration

# Integration test (requires OPENHANDS_API_KEY)
OPENHANDS_API_KEY=sk-oh-... uv run pytest tests/integration/ -v

# E2E test script (live sandbox, ~80s)
OPENHANDS_API_KEY=sk-oh-... uv run python scripts/test_automation.py --api-url https://staging.all-hands.dev
```

## Product Telemetry Identity


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenHands/automation](https://github.com/OpenHands/automation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
