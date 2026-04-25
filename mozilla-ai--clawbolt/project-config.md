---
trigger: always_on
description: Clawbolt is an AI assistant for the trades. FastAPI backend with a Telegram messaging interface and a custom tool-calling agent loop built on any-llm. Built by Mozilla.ai using the open-core model.
---

# Clawbolt

Clawbolt is an AI assistant for the trades. FastAPI backend with a Telegram messaging interface and a custom tool-calling agent loop built on any-llm. Built by Mozilla.ai using the open-core model.

## Build & Run Commands

```bash
# Install dependencies
uv sync

# Run server (requires PostgreSQL -- see docker-compose.yml)
uv run uvicorn backend.app.main:app --reload

# Run with Docker (starts Postgres + app, runs migrations automatically)
docker compose up

# Database migrations
uv run alembic upgrade head
uv run alembic revision --autogenerate -m "description"

# Tests
uv run pytest -v

# Lint & format
uv run ruff check backend/ tests/ alembic/
uv run ruff format --check backend/ tests/ alembic/

# Type checking
uv run ty check --python .venv backend/ tests/ alembic/
```

## Tech Stack

- Python 3.11+, FastAPI, SQLAlchemy 2.0, Pydantic v2
- any-llm-sdk (LLM provider abstraction via `amessages`)
- Telegram Bot API for messaging (via python-telegram-bot)
- Dropbox/Google Drive for file storage
- PostgreSQL for all data persistence, Alembic for migrations
- uv + hatchling build system, ruff linting, ty type checking

## Storage

All structured data is stored in PostgreSQL (configurable via `DATABASE_URL`). The database has 12 tables:

| Table | Purpose |
|---|---|
| `users` | User profiles, personality text, preferences |
| `channel_routes` | Channel -> user routing (Telegram, webchat, etc.) |
| `sessions` | Chat session metadata |
| `messages` | Chat messages (FK to sessions) |
| `media_files` | Media file manifest |
| `memory_documents` | Structured memory and compaction history |
| `heartbeat_logs` | Heartbeat send log |
| `idempotency_keys` | Webhook deduplication |
| `llm_usage_logs` | Token usage tracking |
| `tool_configs` | Per-user tool configuration |
| `calendar_configs` | Per-user calendar integration settings |
| `oauth_tokens` | Encrypted OAuth tokens for integrations (Google Calendar, QuickBooks) |

Key store modules:
- `backend/app/agent/user_db.py` -- `UserStore` (singleton via `get_user_store()`)
- `backend/app/agent/session_db.py` -- `SessionStore` (per-user via `get_session_store(id)`)
- `backend/app/agent/memory_db.py` -- `MemoryStore` (per-user via `get_memory_store(id)`)
- `backend/app/agent/stores.py` -- `MediaStore`, `HeartbeatStore`, `IdempotencyStore`, `LLMUsageStore`, `ToolConfigStore`
- `backend/app/agent/dto.py` -- Pydantic DTOs: `UserData`, `StoredMessage`, `SessionState`, etc.
- `backend/app/agent/file_store.py` -- Compatibility shim (re-exports from above modules)
- `backend/app/database.py` -- `Base`, `SessionLocal`, `get_db()`, `get_engine()`
- `backend/app/models.py` -- All SQLAlchemy ORM model classes

File storage for uploads uses the local filesystem under `data/` (configurable via `DATA_DIR`).

## Backwards Compatibility

Until this project has its first production release, you do not need to be concerned about backwards compatible changes.

## Coding Standards

- All type annotations required
- Ruff rules: `E, F, I, UP, B, SIM, ANN, RUF` (line length 100, `E501` and `B008` ignored)
- SQLAlchemy 2.0 `mapped_column` style for all ORM models
- Pydantic v2 for all data classes and request/response schemas
- All routes `async def`
- All LLM calls via any-llm `amessages` (async)
- Never use `BaseHTTPMiddleware` for streaming endpoints -- use pure ASGI middleware
- Conventional commit prefixes: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `ci:`, `chore:`
- Every data endpoint uses `Depends(get_current_user)` with `user_id` scoping
- Config via Pydantic `BaseSettings` with `extra="ignore"`
- Prefer `isinstance` checks and direct typed attribute access over `getattr`, `hasattr`, or string-based type checks. Our objects are properly typed; using `getattr` with defaults masks real problems when types change. Only use `getattr`/`hasattr` when explicitly directed or at true dynamic boundaries (e.g. plugin APIs).
- Never use em dashes in user-facing content, comments, or copy -- use periods, commas, colons, or pipes instead
- All imports at the top of the file. No inline or deferred imports inside functions. The only exception is `TYPE_CHECKING` guarded imports.

## Testing

- pytest with FastAPI `TestClient`
- PostgreSQL for all tests (requires a local `clawbolt_test` database; see conftest.py)
- `reset_stores()` clears cached store singletons between tests
- Override `get_current_user` via FastAPI dependency injection
- Mock ALL external services: Telegram, LLM (any-llm), faster-whisper, Dropbox/Drive
- Bug fixes must include regression tests

## Architecture

- **PostgreSQL storage**: all structured data in PostgreSQL via SQLAlchemy 2.0 ORM. See `backend/app/database.py` and `backend/app/models.py`. Store modules in `backend/app/agent/` provide CRUD APIs.
- **Auth plugin infrastructure**: base.py (ABC), loader.py (dynamic import), dependencies.py (get_current_user), scoping.py (row-level auth). OSS is single-tenant; premium adds multi-tenant auth via plugin.
- **`user_id` scoping** on every data class and endpoint from day one

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mozilla-ai/clawbolt](https://github.com/mozilla-ai/clawbolt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
