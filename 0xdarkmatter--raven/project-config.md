---
trigger: always_on
description: Guide for AI agents (and human contributors) working inside this codebase.
---

# AGENTS — raven developer guide

Guide for AI agents (and human contributors) working inside this codebase.

## What this project is

**raven** is a SQLite-backed, role-addressable message bus for live coordination between agent sessions. It is the lower-level reusable messaging primitive extracted from [Axiom](https://github.com/0xDarkMatter/axiom).

- **Package name (pip):** `raven`
- **Python import:** `from claude_bus import BusClient` — the import root is `claude_bus`, not `raven`
- **Binary:** `raven` (entry point: `claude_bus.cli.main:cli_main`)
- **Version:** 0.1.1

The distinction matters: `pip install raven` is correct; `import raven` will fail. Always use `from claude_bus import ...`.

## Running tests

```bash
# Full suite (recommended)
python -m pytest tests/ -p no:cacheprovider --tb=short -q

# Unit tests only
python -m pytest tests/unit/ -p no:cacheprovider --tb=short -q

# Integration tests only (run examples as subprocesses)
python -m pytest tests/integration/ -v --tb=short

# Coverage (must stay at 100%)
python -m pytest tests/ --cov=claude_bus --cov-fail-under=100 -q
```

The test suite has 179 tests. All must pass before committing. Coverage is locked at 100% via `pyproject.toml`; new code needs tests.

## Architecture overview

```
src/claude_bus/
├── _core.py          low-level SQLite primitives (send, list_unread, list_since,
│                     try_claim, resolve, read_by_id) — no identity awareness
├── client.py         BusClient: high-level public API wrapping _core
│                     - send() / inbox() / read() / ack() / subscribe()
│                     - auto-registers sender + recipient aliases on every send
├── aliases.py        (role, session_id) ↔ deterministic hex alias
│                     compute_alias() is pure — no DB round-trip
│                     register() is idempotent (INSERT OR IGNORE)
├── db.py             init_db() — idempotent schema apply, process-cached
│                     connection() — context manager, WAL + foreign keys
│                     teardown_session() — row-scoped cleanup
├── schemas.py        SchemaRegistry — opt-in Pydantic body validation per type
│                     permissive by default; strict mode raises SchemaValidationError
├── session.py        register_role_alias() helper
├── exceptions.py     ClaudeBusError hierarchy
├── http.py           Starlette bridge (optional; requires [http] extra)
├── cli/              Typer CLI — 10 commands
│   ├── main.py       entry point + error rendering (cli_main)
│   ├── init.py       raven init
│   ├── doctor.py     raven doctor
│   ├── send.py       raven send
│   ├── inbox.py      raven inbox
│   ├── read.py       raven read
│   ├── ack.py        raven ack
│   ├── tail.py       raven tail  (identity-free observer, never consumes)
│   ├── serve.py      raven serve
│   ├── session.py    raven session init
│   └── _common.py    shared exit codes + output helpers
└── migrations/
    └── 0001_initial.sql   applied idempotently by init_db()
```

### Key invariants

- **Address format:** `<role>:<session>` — role must not contain `:`, neither part may be empty.
- **Aliases are deterministic:** `compute_alias(role, session)` always returns the same hex string. Producers can address recipients that haven't booted yet.
- **At-most-once delivery under `subscribe()`:** `try_claim()` does `UPDATE … WHERE status IN ('sent','delivered')`. The winner gets rowcount=1; everyone else gets 0 and skips. No locking beyond SQLite's WAL.
- **At-least-once under `inbox()`:** messages stay `sent` until explicitly acked. Call `ack(id)` after processing.
- **`tail` never consumes:** `list_since()` queries by id range, never changes any status. Safe to run alongside active consumers.
- **`init_db()` is process-cached:** it only runs the migration SQL once per process per DB path. Pass `force=True` in tests that need a fresh DB.

### SQLite schema

Two tables: `aliases` and `messages`.

```
aliases:  alias (PK), role, session_id, created_at
messages: id (PK, autoincrement), session_id, sender, recipient, type, urgency,
          body (JSON), tags, in_reply_to (FK), conversation_id (FK),
          ref_id, task_id, status, expires_at, created_at, delivered_at, resolved_at
```

`conversation_id` is the raw DB column for what `BusClient.Message` exposes as `correlation_id`. Use `conversation_id` in raw SQL (e.g. integration tests); use `correlation_id` when working through the public API.

## Public Python API

```python
from claude_bus import (
    BusClient,
    Message,
    SchemaRegistry,
    ClaudeBusError,
    SchemaValidationError,
    UnknownMessageError,
    init_db,
    register_role_alias,
)
```

### BusClient

```python
client = BusClient(session_id="swarm-1", role="conductor", db_path="bus.db")

# Send
msg = client.send(
    to="architect:swarm-1",
    type="plan",
    body={"step": 1},
    correlation_id=42,   # optional — threads this message to another
    reply_to=10,         # optional — in_reply_to a specific message id
    tags=["urgent"],
    urgency="critical",
    expires_in_s=300,
)

# Read inbox (unread messages for this role)
messages = client.inbox(max=50)

# Read a specific message by id (no status change)
msg = client.read(message_id=1)

# Acknowledge (mark read)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xDarkMatter/raven](https://github.com/0xDarkMatter/raven) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
