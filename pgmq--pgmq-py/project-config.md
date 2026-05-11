---
trigger: always_on
description: This file contains project-specific context for AI coding agents. The project is a Python client library for the [PGMQ](https://github.com/pgmq/pgmq) PostgreSQL extension.
---

# AGENTS.md — pgmq-py

This file contains project-specific context for AI coding agents. The project is a Python client library for the [PGMQ](https://github.com/pgmq/pgmq) PostgreSQL extension.

---

## Project Overview

`pgmq` is the official Python client for PGMQ (Postgres Message Queue). It exposes synchronous and asynchronous APIs, with both raw driver (psycopg/asyncpg) and SQLAlchemy-based backends. The public API surface is intentionally identical across all four client variants so users can swap implementations with minimal changes.

- **Package name:** `pgmq`
- **Version:** `1.0.6` (pyproject.toml) — note that `src/pgmq/__init__.py` hard-codes `__version__ = "1.1.0"`; keep these in sync when bumping.
- **License:** Apache-2.0
- **Python support:** `>=3.9`
- **Repository:** https://github.com/pgmq/pgmq-py

### Technology Stack

| Layer | Choices |
|-------|---------|
| Build / package manager | `uv` (build backend `uv_build`) |
| Sync driver | `psycopg[binary,pool]>=3.2.10` |
| Async driver | `asyncpg>=0.30.0` (optional extra `[async]`) |
| SQLAlchemy | `sqlalchemy>=2.0.0` (optional extras `[sqlalchemy]` / `[sqlalchemy-async]`) |
| JSON | `orjson>=3.11.3` |
| Lint / format | `ruff>=0.12.12` |
| Logging | stdlib `logging` with optional `loguru` fallback |
| Benchmarks | `locust`, `pandas`, `scipy`, `typer` |
| Documentation | `mkdocs`, `mkdocs-material`, `mkdocstrings`, `mike` |

---

## Project Structure

```
src/pgmq/
  __init__.py               # Public exports, backward-compat aliases, version
  base.py                   # PGMQConfig dataclass + BaseQueue (shared init/logging)
  queue.py                  # Sync psycopg-based PGMQueue
  async_queue.py            # Async asyncpg-based PGMQueue
  sqlalchemy_queue.py       # Sync SQLAlchemy-based PGMQueue
  sqlalchemy_async_queue.py # Async SQLAlchemy-based PGMQueue
  _sql.py                   # All SQL templates + conversion helpers (%s → $N, %s → :param_N)
  messages.py               # Dataclasses mapping PGMQ composite types (Message, QueueMetrics, ...)
  decorators.py             # Transaction decorators (transaction, async_transaction, sqlalchemy_transaction, sqlalchemy_async_transaction)
  logger.py                 # LoggingManager with dual stdlib/loguru backend
  notify_listener.py        # SyncNotificationListener + AsyncNotificationListener (PostgreSQL NOTIFY/LISTEN)

tests/
  utils.py                  # PGMQTestCase base class + env-driven PG_* constants
  test_integration.py       # Sync psycopg integration tests
  test_async_integration.py # Async asyncpg integration tests
  test_sqlalchemy_integration.py       # Sync SQLAlchemy integration tests
  test_sqlalchemy_async_integration.py # Async SQLAlchemy integration tests
  test_features.py          # Partitioning, notifications, validation utilities
  test_routing.py           # Topic routing (bind/send/unbind/test_routing)
  test_notify_listener.py   # NOTIFY/LISTEN tests for all four backends
  test_sql_conversion.py    # Pure unit tests for _sql.py conversions (no DB required)
  test_logger.py            # Logger unit tests

example/
  example_app_sync.py       # Transaction decorator usage examples
  example_app_async.py      # Async transaction usage examples

benches/
  bench.py / runner.py / ... # Locust-based load testing (dependency-group "bench")

docs/
  index.md                    # Documentation homepage
  getting_started.md          # Installation & quick start
  configuration.md            # PGMQConfig reference
  clients.md                  # Four backend clients
  queue_management.md         # Create, drop, list, purge queues
  messages.md                 # Dataclasses (Message, QueueMetrics, ...)
  sending_messages.md         # send, send_batch, headers, delay
  reading_messages.md         # read, read_with_poll, FIFO variants
  deleting_and_archiving.md   # delete, archive, pop, purge
  visibility_timeout.md       # set_vt
  topic_routing.md            # Topic-based routing
  metrics.md                  # Queue statistics
  notifications.md            # NOTIFY/LISTEN & listeners
  transactions.md             # Decorators & manual transactions
  logging.md                  # Logging configuration
  utilities.md                # Validation, FIFO indexes
  backward_compatibility.md   # Migration notes
  development.md              # Tests, contributing, MkDocs/Mike

mkdocs.yml                    # MkDocs configuration (Material theme, Mike versioning)
```

---

## Build, Install, and Test Commands

All common tasks are wrapped in the `Makefile`:

```bash
# Install everything (dev + all extras + bench)
uv sync --all-groups --all-extras

# Format code and auto-fix lint issues
make format
#  → uv run ruff format src/
#  → uv run ruff check --fix --exit-zero src/

# Run lint checks without modifying files
make lint
#  → uv run ruff check src/
#  → uv run ruff format --check src/

# Run the full test suite (spins up Docker PostgreSQL automatically)
make test
#  → docker rm -f pgmq-postgres
#  → docker run -d --name pgmq-postgres -e POSTGRES_PASSWORD=postgres -p 5432:5432 ghcr.io/pgmq/pg18-pgmq:latest
#  → sleep 10

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pgmq/pgmq-py](https://github.com/pgmq/pgmq-py) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
