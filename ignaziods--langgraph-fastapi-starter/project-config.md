---
trigger: always_on
description: This file defines how Claude Code operates within this repository.
---

# CLAUDE.md — langgraph-fastapi-starter

This file defines how Claude Code operates within this repository.
Read it completely before touching any file. It governs every decision.

---

## What This Project Is

A production-grade starting point for AI agent backends. Not a demo. Not a tutorial.
A template that a technical founder can fork, configure, and extend into a real product
in under 30 minutes.

The constraint that shapes everything: **minimum surface area for maximum production readiness.**
Every file that exists must earn its place. Every abstraction must be removable without
breaking the system. Every decision must be documented so the person forking this can
disagree with it and change it confidently.

---

## The One Rule

**This is a starter, not a framework.**

A framework hides decisions. A starter exposes them.

When you are unsure whether to add something, ask: "Does a founder need to understand this
to build their product?" If yes, it belongs. If it's infrastructure they'd never touch,
it probably doesn't.

---

## Architecture Principles

**FastAPI + LangGraph + pgvector + Alembic. That is the stack. Do not extend it.**

No Redis. No Celery. No Kafka. No additional databases. No service mesh.
If something requires one of these to work, it does not belong in the starter.
The founder can add them when they need them. The starter should not impose them.

**One agent, one graph.**
The starter ships with a single example agent: a research assistant that uses web search
and document retrieval. It exists to show how the graph is wired, not to be production-ready
itself. It will be deleted and replaced by whoever forks this. Make it clear, not clever.

**Alembic for all schema changes. No exceptions.**
No CREATE TABLE in application startup code. No schema changes in seed scripts.
All database structure lives in migrations/. The startup sequence is:
alembic upgrade head → seed (if needed) → run.

**Sync database access, async FastAPI.**
Use psycopg2 (sync) for database operations inside a thread pool executor.
FastAPI is async; database calls are sync and offloaded via `run_in_executor`.
Do not introduce SQLAlchemy async. Do not introduce asyncpg. The complexity is not
worth it at this stage. Document this decision in CLAUDE.md.

**Structured logging, not print statements.**
Every log entry is JSON. Fields: timestamp, level, request_id, event, and any
relevant context. Use Python's logging module with a JSON formatter. No third-party
logging libraries. The founder will replace this with their preferred stack
(Datadog, Plex, whatever) — keep it swappable.

**API key auth, nothing more.**
Bearer token in Authorization header. Keys stored as bcrypt hashes in postgres.
One table: api_keys (id, key_hash, name, tenant_id, role, created_at, last_used_at, revoked_at).
No OAuth. No JWT. No sessions. If a founder needs OAuth, they will add it.
The starter gives them the hook (a middleware function) but not the complexity.

---

## Code Standards

**Type hints are required on every function.** Return types included.
`Any` is banned. If you need `Any`, the data model is wrong.

**Pydantic v2 for all request/response models.**
No TypedDict for API boundaries. No raw dicts. Every endpoint has a typed request model
and a typed response model, even if trivial.

**Error handling is explicit.**
Every endpoint has a try/except with specific exception types.
HTTP errors use FastAPI's HTTPException with a code and a message that a developer
can act on. No `except Exception as e: return 500`.

**No global state outside of config.**
The database connection pool, the LangGraph graph instance, and the embedder are
initialized at startup via FastAPI lifespan events and passed via dependency injection.
Nothing is instantiated at module import time except the config singleton.

**Tests are real assertions.**
Integration tests hit the actual FastAPI app via httpx.AsyncClient.
Unit tests assert exact values, not just type or length.
No test that only asserts `response.status_code == 200` without also checking the body.

---

## File Conventions

**app/main.py** — FastAPI app factory only. Lifespan events, middleware registration,
router inclusion. No business logic. No database queries. No LangGraph calls.

**app/routers/** — One file per resource: agents.py, api_keys.py, health.py.
Routers contain endpoint definitions only. No SQL. No graph calls directly.
Delegate to services/.

**app/services/** — Business logic. agent_service.py calls the graph.
api_key_service.py manages key lifecycle. These are plain Python classes.
No FastAPI imports in services/.

**app/graph/** — LangGraph definitions. graph.py builds and compiles the graph.
nodes.py contains individual node implementations. state.py defines the TypedDict state.
tools.py defines tools. Nothing in graph/ should import from routers/ or services/.

**app/db/** — Database layer. connection.py manages the psycopg2 pool.
queries.py contains raw SQL as module-level constants (not f-strings — parameterized).
No ORM. No query builder. Named parameters only.

**app/middleware/** — auth.py for API key validation. logging.py for request logging.
Both are FastAPI middleware, not dependencies (they run on every request).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IgnazioDS/langgraph-fastapi-starter](https://github.com/IgnazioDS/langgraph-fastapi-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
