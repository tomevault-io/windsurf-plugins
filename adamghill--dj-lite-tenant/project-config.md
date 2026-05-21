---
trigger: always_on
description: - **Type-checking:** `ty` — run with `uv run ty check`
---

## Tooling

- **Type-checking:** `ty` — run with `uv run ty check`
- **Linting / formatting:** `ruff` — run with `uv run ruff check` and `uv run ruff format`
- **Arbitrary style rules:** `woof` enforces the project's custom formatting conventions deterministically — run with `uvx --from git+https://github.com/adamghill/woof woof format`

Always run all three before committing.

---

## Project Overview

`dj-lite-tenant` gives each tenant (by default, each Django `User`) their own isolated SQLite database. Tenant databases are created automatically on first save and routed transparently via Django's database router. Status: **experimental, not production-ready**. Forked from [django-sqlite-user-db](https://github.com/MessyComposer/django-sqlite-user-db).

- Python ≥ 3.11, Django ≥ 5.1
- Dependencies: `dj-lite` (SQLite config helpers), `typeguard` (runtime type checking)
- Build: `hatchling`, packaged with `uv`
- Tests: `pytest` + `pytest-django` + `pytest-mock`

---

## Architecture & Data Flow

```
HTTP request
  └─ TenantDatabaseMiddleware
       ├─ calls TENANT_ID_CALLABLE(request) → tenant_pk (str | None)
       ├─ sets _current_tenant_pk ContextVar
       └─ calls get_or_create_tenant_db(tenant_pk)
            ├─ registers alias in settings.DATABASES
            └─ connection_registry.touch(alias)  [LRU eviction]

ORM query
  └─ TenantDatabaseRouter
       ├─ checks is_tenant_app_or_model(app_label, model_name)
       ├─ if tenant model → get_current_tenant_pk() → get_tenant_db_alias()
       └─ else → "default"

New connection opened (signal)
  └─ on_connection_created → attach_shared_databases()
       └─ ATTACH DATABASE ... AS "shared"  (enables cross-DB ORM joins)
```

For out-of-request use, the `tenant_db(obj)` context manager sets the ContextVar and tears it down on exit.

---

## Module Responsibilities

| File | Responsibility |
|---|---|
| `conf.py` | Read `DJ_LITE_TENANT` settings with defaults; `get_conf()`, `get_tenant_model()`, `get_tenant_id_callable()`, `is_tenant_app_or_model()` |
| `middleware.py` | `TenantDatabaseMiddleware` (sets ContextVar per-request); `tenant_db()` context manager; `get_tenant_pk_from_request()` default callable |
| `routers.py` | `TenantDatabaseRouter` — routes reads/writes/migrations to the right DB alias |
| `utils.py` | DB file lifecycle: `get_or_create_tenant_db`, `setup_tenant_db`, `close_tenant_db`, `delete_tenant_db`, `clear_template_cache`, `attach_shared_databases`, `get_attach_statements` |
| `connection_registry.py` | Thread-safe LRU registry (`OrderedDict`) tracking open tenant aliases; evicts least-recently-used when over `MAX_OPEN_CONNECTIONS`; includes `clear()` method for bulk cleanup |
| `signals.py` | `create_tenant_database` (post_save), `delete_tenant_database` (post_delete), `on_connection_created` (ATTACH), `invalidate_template_cache` (post_migrate) |
| `apps.py` | `DjLiteTenantConfig.ready()` wires `post_save`/`post_delete` signals to the tenant model |
| `admin.py` | `SwitchTenantAdmin` mixin; `set_tenant` / `unset_tenant` views for superuser DB switching |
| `urls.py` | URL patterns for `set_tenant` / `unset_tenant` admin views (namespace `dj_lite_tenant`) |
| `backends/sqlite3/base.py` | Custom `DatabaseWrapper` using `DatabaseOperations` below |
| `backends/sqlite3/operations.py` | Overrides `quote_name` to emit `"schema"."table"` for dotted `Meta.db_table` names — required for ATTACH'd DB ORM queries |
| `management/commands/create_tenant_db.py` | `create_tenant_db <tenant_pk>` — creates/migrates a single tenant DB |
| `management/commands/migrate_tenant_dbs.py` | `migrate_tenant_dbs` — applies migrations to all existing tenant DB files in `DIR` |

---

## Key Design Decisions

- **ContextVar for tenant isolation** — `_current_tenant_pk` is a `ContextVar`, giving per-async-task / per-thread isolation with no globals.
- **Dynamic `settings.DATABASES`** — tenant DB aliases are registered at runtime, not at startup. The alias format is `tenant_<slugified_pk>`.
- **LRU connection eviction** — `connection_registry` keeps an `OrderedDict` sorted by last access. When `len > MAX_OPEN_CONNECTIONS`, the oldest alias is closed and removed from `settings.DATABASES`.
- **ATTACH DATABASE for cross-DB joins** — on every new tenant connection, the shared DB is ATTACHed as `"shared"` (configurable via `ATTACHMENTS`). The custom backend's `quote_name` splits `"shared.tablename"` → `"shared"."tablename"` so the ORM handles it correctly. Attachments are opened in read-only mode to prevent write conflicts and are restricted by SQLite's SQLITE_LIMIT_ATTACHED configuration (default 10).
- **Template-based provisioning** — when `USE_DATABASE_TEMPLATE=True`, the first migrated tenant DB is copied to `.template.sqlite3`; subsequent tenants are created by file copy (fast). The template is invalidated on `post_migrate` for any tenant app.
- **`typechecked`** — functions use `@typeguard.typechecked` for early, clear errors on misconfiguration.
- **`DELETE_TENANT_DB_ON_DELETE` defaults to `False`** — safety default to prevent accidental data loss; logs a warning when a tenant is deleted but the file remains.

---

## `DJ_LITE_TENANT` Settings Reference

| Key | Default | Description |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adamghill/dj-lite-tenant](https://github.com/adamghill/dj-lite-tenant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
