---
trigger: always_on
description: **MyBibliotheca** is a self-hosted personal library and reading tracker built with Flask and KuzuDB graph database. It provides an open-source alternative to Goodreads and StoryGraph for managing your personal book collection, tracking reading progress, and generating reading statistics.
---

# MyBibliotheca – GitHub Copilot Instructions

## About This Repository

**MyBibliotheca** is a self-hosted personal library and reading tracker built with Flask and KuzuDB graph database. It provides an open-source alternative to Goodreads and StoryGraph for managing your personal book collection, tracking reading progress, and generating reading statistics.

**Key Technologies:** Python 3.13+, Flask, KuzuDB (graph database), Bootstrap UI, Docker

## AI Engineering Guide

Concise, project-specific rules so an agent can contribute safely and fast. Focus on Kuzu graph patterns, lazy services, and schema preflight.

### Pattern Example
```python
def sync_method(self): return run_async(self.async_method())
```

### 1. Core Architecture (Graph + Services)
* Single KuzuDB instance (no horizontal scaling). Always assume WORKERS=1.
* Schema is created/augmented in `app/infrastructure/kuzu_graph.py::_initialize_schema()`; only additive changes are automatic unless `KUZU_FORCE_RESET=true`.
* Domain-first: entities in `app/domain/models.py` (dataclasses, enums for ReadingStatus, OwnershipStatus, etc.). Keep persistence concerns out of domain models.
* Service layer: lazy singletons exposed in `app/services/__init__.py` (e.g. `from app.services import book_service, user_service`). First attribute access triggers initialization and optional one-time migrations.
* Backward compatibility: `KuzuServiceFacade` is the abstraction for book-related operations during migration—prefer calling through facade unless extending a clearly separated service.

### 2. Service & Async Pattern
* Provide both async + sync variants: implement `async def _op(...)` then thin sync wrapper using `run_async` from `kuzu_async_helper`.
* Do NOT open new DB connections inside services; rely on existing singleton helpers (search for existing patterns before adding new factories).
* When adding a new service: place file in `app/services/`, expose a lazy getter + `_LazyService` entry in `services/__init__.py`, and add to `__all__`.

### 3. Schema & Data Evolution
* Add new node/rel properties by editing creation lists in `_initialize_schema()`; for existing deployments, code defensively: attempt access → on failure issue `ALTER TABLE ... ADD` (see Person / ReadingLog upgrade examples).
* Never drop/rename automatically. Write explicit one-off migration helpers or guarded env-flag paths.
* Use `KUZU_DEBUG=true` to surface file and schema introspection logging; `KUZU_FORCE_RESET=true` only for destructive resets (warn loudly in PRs).
* Master schema preflight: `app/schema/master_schema.json` (if present) is treated as declarative expectation. Startup preflight (invoked early via `schema_preflight_state` files) computes missing columns / relationships and applies only additive operations (CREATE / ALTER ADD). Environment flags: `DISABLE_SCHEMA_PREFLIGHT`, `PREFLIGHT_REL_ONLY`, `PREFLIGHT_NODES_ONLY`, `SKIP_PREFLIGHT_BACKUP`. A backup is created unless skipped. After successful application expect log line: `Schema preflight upgrade complete`.
* When introducing new functionality that needs columns: (1) Add to `master_schema.json` (increment version) (2) Add same column in `_initialize_schema()` create list (ensures greenfield DBs match) (3) Optionally add a defensive lazy `ALTER` try/except near first access to smooth upgrades if preflight was disabled.
* Avoid logic that assumes column presence without fallback; pattern: try query referencing column → on exception containing `Cannot find property <col>` → run `ALTER TABLE <Node> ADD <col> TYPE` → retry.

### 4. Graph Usage & Integrity
* Prefer relationship-based queries (e.g., user isolation via traversing OWNS rather than filtering raw properties).
* When deleting nodes with edges, use `DETACH DELETE` (or explicit pattern) to avoid orphaned rels.
* Maintain bidirectional semantics explicitly: e.g., `WRITTEN_BY` (Book→Person) plus `AUTHORED` (Person→Book). Follow established naming if adding new directional pairs.

### 4a. Concurrency & Safe Kuzu Access
* Use `SafeKuzuManager` (`app/utils/safe_kuzu_manager.py`) for thread-safe connection-per-operation access—do NOT resurrect raw global singletons.
* Pattern: `from app.utils.safe_kuzu_manager import safe_execute_query, safe_get_connection`.
	```python
	from app.utils.safe_kuzu_manager import safe_execute_query, safe_get_connection
	rows = safe_execute_query("MATCH (b:Book) RETURN COUNT(b) AS c", operation="count_books")
	with safe_get_connection(operation="batch_import") as conn:
			conn.execute("CREATE (t:Temp {id: $id})", {"id": some_id})
	```
* Each call acquires lock → initializes DB once → creates short‑lived `kuzu.Connection` → closes deterministically; this lowers corruption risk while allowing moderate intra-process concurrency (threads / Flask requests).
* For multi-query sequences needing consistency, wrap in one `with safe_get_connection(...)` rather than multiple helper calls to minimize lock churn.
* Backups: acquire quiesced window via `quiesce_for_backup()` (see `SimpleBackupService`)—avoid ad hoc pausing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pickles4evaaaa/mybibliotheca](https://github.com/pickles4evaaaa/mybibliotheca) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
