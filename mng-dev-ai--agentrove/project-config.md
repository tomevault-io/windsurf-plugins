---
trigger: always_on
description: - `frontend/backend-sidecar/` is a build artifact — never edit; all backend source lives in `backend/`
---

# CLAUDE.md

## Project Context

- `frontend/backend-sidecar/` is a build artifact — never edit; all backend source lives in `backend/`
- Open-source self-hosted app for single-user / small-team use — single API instance, no distributed workers or multi-replica coordination
- No distributed-system patterns (distributed locks, cross-instance heartbeats, consensus) — prefer in-process state (in-memory sets/dicts, asyncio tasks)
- Redis is for pub/sub and caching only — not a task broker or coordination layer
- Background work runs as asyncio tasks in the API process
- Treat per-user request handling as effectively sequential — don't flag bugs that only appear under overlapping concurrent requests (retries, double-submit, multi-tab) unless the task explicitly asks for concurrency hardening
- ACP `field_meta` (`_meta`) is extensibility metadata agents aren't required to read — don't use it for user-facing data (alt instructions, form answers); if ACP has no first-class field for a concept, it can't be reliably done through metadata

## SQLAlchemy Model Conventions

- Always pair `default=` with `server_default=` — `default` only applies in Python/ORM
- Always specify `nullable=True|False` explicitly
- Always set max length on String fields (e.g., `String(64)`)
- Use `DateTime(timezone=True)` for all datetime fields
- Don't add `index=True` on an FK if a composite index already starts with that column

## Migration Workflow

- Generate migrations via Alembic autogenerate (don't write them by hand); manual edits to generated migrations are fine when needed for correctness
- Run Alembic commands inside the Docker backend container (not on host)

## Code Style

### Minimalism
- Choose the smallest fix — don't refactor, restructure, or add abstractions as part of a bug fix; prefer a one-line guard over reworked control flow
- Don't optimize for "no regressions" or long-term resilience unless asked — favor simple, direct changes over defensive scaffolding
- Don't build elaborate rollback/state-restoration for failure paths — log + best-effort recovery (e.g., re-queue) is sufficient; don't save/restore every field, delete orphaned rows, or revert intermediate changes
- Don't add resource cleanup (`try/finally` with `.cleanup()`, `.close()`) for short-lived provider/client objects — GC handles lazy clients (e.g., `aiodocker.Docker`); only add cleanup for long-lived or pooled objects
- Don't add pre-flight compatibility checks when a natural fallback exists — let it fall through to the existing path instead of branching to re-route
- Validate at the boundary only — if an API endpoint checks a value, downstream functions receiving it shouldn't re-validate
- Prefer the simplest collection op (e.g., `list.insert(0, item)` over a sorted-insertion loop when order doesn't matter)
- Don't add backward-compat paths, fallback paths, or legacy shims unless asked
- Don't create type aliases with no semantic value (e.g., `StreamKind = str`)
- Don't handle hypothetical input shapes — code for the format you've observed (logs, tests, types), not branches for unseen structures
- Avoid no-op pass-through wrappers; wrappers must add concrete value (validation, transformation, error handling, compatibility boundary, stable public API); prefer direct imports/calls otherwise
- Don't extract a utility file for a single constant/expression duplicated across only 2 call sites — inline until 3+ sites or an existing file fits naturally
- Don't create standalone functions that only wrap a single dict lookup with a default — inline `DICT[key].field` or `DICT.get(key)`; for tuples, use a `NamedTuple` instead of accessor functions
- Don't create inline dict literals for identity mappings — use a module-level `frozenset` membership check

### Exceptions
- Keep `try/except` narrow — wrap only the code needing the specific recovery; code that can safely propagate should stay outside the `try`
- Narrow `except` clauses to specific types — never `except Exception` when failure modes are known
- Don't translate exceptions across boundaries just to change the type — let meaningful upstream errors propagate; catch-and-wrap only when the caller needs a different status/shape
- When catching a `ServiceException` subclass at the API boundary, use `exc.status_code` — don't hardcode a status that shadows the exception's classification
- When a function receives an optional targeting parameter (e.g., `cwd`, `workspace_id`) and the value is invalid, raise — don't silently fall back to a default target

### Input & Security
- Don't use Python `str.format()` or f-strings to interpolate untrusted content that may contain `{`/`}` (diffs, code, JSON) — use concatenation or `string.Template`
- Add `Field(max_length=...)` to all `str` fields on Pydantic request models; add `min_length=1` when empty is invalid

### FastAPI
- Don't instantiate services in route handlers — add a factory in `deps.py` and inject via `Depends()`; route files shouldn't import `SessionLocal`
- Don't define helper functions in endpoint files — endpoint files contain only route handlers. Inline no-op exception-translation wrappers at the call site; move reusable access/service helpers to `deps.py`; move pure utilities to `utils/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mng-dev-ai/agentrove](https://github.com/Mng-dev-ai/agentrove) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
