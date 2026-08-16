---
trigger: always_on
description: SQLAlchemy-Continuum is a versioning and auditing extension for SQLAlchemy:
---

# CLAUDE.md

SQLAlchemy-Continuum is a versioning and auditing extension for SQLAlchemy:
every flush of a versioned model writes a row to a `<table>_version` table
tied to a `transaction` record. This repo is the maintained fork published to
PyPI as `SQLAlchemy-Continuum` (GitHub org `sqlalchemy-continuum`).

Supports Python >= 3.10 and SQLAlchemy >= 1.4.53, < 2.1 — code must work on
BOTH the 1.4 and 2.x lines (use the 1.4 "future" API: `sa.select()`,
`session.get()`, `scalar_subquery()`, `session.scalars()`).

## Commands

```bash
uv venv && uv pip install -e . --group dev   # setup
DB=sqlite uv run pytest                      # tests (sqlite is the default DB)
DB=postgres uv run pytest                    # needs local postgres (see tests/__init__.py for URLs)
uv run pytest tests/test_insert.py -q        # single module
uv run ruff check --fix && uv run ruff format
uv run tox                                   # full py × SQLAlchemy matrix
uv run zensical serve                        # docs live preview (http://localhost:8000)
uv run zensical build --clean --strict       # docs build into site/
```

`DB` selects the backend: `sqlite` (default), `postgres`, `postgres-native`
(same postgres URL but enables trigger-based native versioning), `mysql`.
`DATABASE_URL` overrides everything.

## Architecture

Configuration time (driven by mapper events, orchestrated by `builder.py`):

- `make_versioned()` (`__init__.py`) registers mapper/session/engine listeners
  on the global singleton `versioning_manager` (`manager.py:VersioningManager`).
- Declaring a model with `__versioned__ = {}` queues it in
  `manager.pending_classes` via the `instrument_class` event.
- On `after_configured` (guarded by `prevent_reentry`), `Builder` builds: the
  version tables (`table_builder.py`), the `Transaction` model
  (`transaction.py:TransactionFactory`), the version model classes
  (`model_builder.py`, generates `<Name>Version` classes inheriting
  `version.py:VersionClassBase`), and relationship reflection
  (`relationship_builder.py`).

Runtime (per flush, orchestrated by `unit_of_work.py:UnitOfWork`):

- `before_flush`: UoW creates a second Session (`version_session`) bound to
  the same connection, plus the `Transaction` row.
- Mapper `after_insert/update/delete` events collect `Operation` objects
  (`operation.py`).
- `after_flush`: `make_versions()` writes one version row per (class,
  identity, transaction) and, under the default `validity` strategy, closes
  the previous version's `end_transaction_id`.
- `commit`/`rollback`: `manager.clear()` discards the UoW (skipped inside
  nested transactions/savepoints).

Support modules: `fetcher.py` (previous/next/index/`version_at`/`all_versions`
queries; `ValidityFetcher` vs `SubqueryFetcher` per the `strategy` option),
`reverter.py` (`version.revert()`), `utils.py` (public helper API),
`schema.py` (offline migration helpers), `_compat.py` (vendored
SQLAlchemy-Utils subset — keep it dependency-free), `dialects/postgresql.py`
(PostgreSQL-only trigger-based "native versioning": plpgsql audit functions,
hstore-based change detection, temp-table transaction-id plumbing),
`plugins/` (hook-based extensions; see `plugins/base.py` for the hook set).

## Hard-won gotchas

- **UnitOfWork is keyed by Connection, not Session** (`manager.units_of_work`).
  SQLAlchemy hands listeners connection *clones*; `_uow_from_conn` matches by
  underlying DBAPI connection. Don't break identity-based matching.
- **Recursion guards are load-bearing**: both `process_before_flush` and
  `process_after_flush` early-return when `session == uow.version_session`;
  `Builder.configure_versioned_classes` is wrapped in `prevent_reentry`.
- **`Operation.INSERT/UPDATE/DELETE` = 0/1/2 are persisted in the DB** and
  hard-coded into relationship SQL (`operation_type != 2`). Never renumber.
- **Version tables have no foreign keys.** Joined-table inheritance uses a
  hand-built `inherit_condition`; version-class "relationships" are computed
  Python properties, not mapper relationships — `joinedload` won't work.
- **The "useless" loop in `TableBuilder.__call__` is not dead code**:
  constructing `sa.Index` with bound columns attaches it to the table as a
  side effect; iterating `_build_composite_indexes(table)` with `pass` is how
  the composite indexes get created.
- **`table_name` option is a printf template** (`'%s_version'`), applied with
  the `%` operator in several places. This format is public API — don't
  convert to f-strings/`.format`.
- Delete-then-insert of the same row in one transaction is recorded as an
  UPDATE (`Operations.add_insert`). Updates whose only changes are
  one-to-many/many-to-many collection state produce no version row.
- Single-table inheritance is special-cased: children extend the parent's
  version table (`extend_existing=True`) and skip the internal columns.
- The SQL DDL templates in `dialects/postgresql.py` and
  `transaction.py:procedure_sql` use `.format()` with named placeholders —
  leave them as `.format` calls.
- Native versioning silently skips writes that don't go through a Continuum
  session (the trigger swallows the missing temp-table exception) and
  requires the `hstore` extension.

## Tests


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sqlalchemy-continuum/sqlalchemy-continuum](https://github.com/sqlalchemy-continuum/sqlalchemy-continuum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
