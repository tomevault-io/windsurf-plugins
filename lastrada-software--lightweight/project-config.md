---
trigger: always_on
description: Lightweight is a thin, modern **C++23 ODBC SQL API** for raw and high-level database access. It exposes a low-level layer (`SqlConnection`, `SqlStatement`, `SqlDataBinder<T>`) and a high-level `DataMapper` (record/relationship mapping). All public symbols live in the `Lightweight` namespace (alias `Light`).
---

# Lightweight — Agent Guidelines

## Project Architecture

Lightweight is a thin, modern **C++23 ODBC SQL API** for raw and high-level database access. It exposes a low-level layer (`SqlConnection`, `SqlStatement`, `SqlDataBinder<T>`) and a high-level `DataMapper` (record/relationship mapping). All public symbols live in the `Lightweight` namespace (alias `Light`).

Per-DBMS differences are funneled through a single dispatch point: `SqlQueryFormatter`. Business logic must be database-agnostic and let the formatter shape the SQL. Currently supported databases:

| Database | `SqlServerType` | Test env name (`.test-env.yml`) |
|----------|-----------------|---------------------------------|
| Microsoft SQL Server  | `MICROSOFT_SQL` | `mssql2017`, `mssql2019`, `mssql2022`, `mssql` |
| PostgreSQL            | `POSTGRESQL`    | `postgres`                       |
| SQLite3               | `SQLITE`        | `sqlite3`                        |

Bundled tools (in `src/tools/`): `dbtool` (general DB CLI), `ddl2cpp` (schema → C++ records generator), `large-db-generator` (test data).

## Component Map

| Path | Purpose |
|------|---------|
| `src/Lightweight/SqlConnection.{hpp,cpp}` | ODBC connection (driver, attrs, transaction, server-type detection) |
| `src/Lightweight/SqlStatement.{hpp,cpp}`  | Prepared statement, bind/execute/fetch, batched execute |
| `src/Lightweight/SqlDataBinder.hpp`       | Generic `SqlDataBinder<T>` traits — specialize per type |
| `src/Lightweight/DataBinder/`             | Binder specializations: strings, GUID, date/time, binary, numeric, variant, optional |
| `src/Lightweight/DataMapper/`             | High-level mapper: `Field`, `BelongsTo`, `HasMany`, `HasManyThrough`, `HasOneThrough`, connection pool, query builders |
| `src/Lightweight/Async/`                  | C++23 coroutine layer: `Task`, `SyncWait`, executors (thread-pool/strand/manual), offload backend, `AsyncSqlTransaction`. Async methods are added to `SqlConnection`/`DataMapper`/`Pool` — a first-class, always-built part of the library. See `docs/async.md` |
| `src/Lightweight/QueryFormatter/`         | Per-DBMS formatters: `SqlServerFormatter.hpp`, `PostgreSqlFormatter.hpp`, `SQLiteFormatter.hpp` |
| `src/Lightweight/SqlQuery/`               | DSL: `Select`, `Insert`, `Update`, `Delete`, `Migrate`, `MigrationPlan`, `Core` |
| `src/Lightweight/SqlMigration.{hpp,cpp}`  | Migration runner |
| `src/Lightweight/SqlScopedLock.{hpp,cpp}` | RAII cross-process advisory lock (used by migrations and by user code) |
| `src/Lightweight/SqlAdvisoryLock.hpp`     | Dialect-handler interface for the advisory lock; one impl per backend |
| `src/Lightweight/SqlBackup/` (+ `.hpp`)   | Schema/data backup framework |
| `src/Lightweight/Tools/`                  | Shared internal tools/utilities used by `src/tools/` binaries |
| `src/Lightweight/Zip/`                    | libzip wrapper used by backup |
| `src/Lightweight/Lightweight.cppm`        | C++20 module export aggregator (when `LIGHTWEIGHT_BUILD_MODULES=ON`) |
| `src/tests/`                              | Catch2 test suite (built as `LightweightTest`); `.test-env.yml` consumed via `--test-env=<name>` |
| `src/tools/`                              | `dbtool/`, `ddl2cpp.cpp`, `large-db-generator/`, `test_chinook.sh` |
| `src/examples/`                           | Compilable usage samples + Chinook integration; `todo/` shows the async API driven through stdexec senders (`Async::AsSender`) |
| `src/benchmark/`                          | Performance benchmarks |
| `cmake/`                                  | `ClangTidy.cmake`, `Coverage.cmake`, `PedanticCompiler.cmake`, `Sanitizers.cmake`, `Version.cmake`, `Lightweight-config.cmake.in` |
| `docs/`                                   | User-facing documentation (`data-binder.md`, `dbtool.md`, `how-to.md`, `sql-migrations.md`, `sqlquery.md`, `usage.md`, `best-practices.md`, `sql-backup-format.md`) |
| `scripts/tests/docker-databases.py`       | Docker harness for MSSQL/Postgres test containers |
| `.github/workflows/build.yml`             | Authoritative CI matrix (presets × databases × sanitizers) |
| `.agent/`                                 | Agent deep-dives (architecture, databases, testing, C++ patterns) |

See `.agent/architecture.md` for a deeper structural walkthrough with canonical examples.

## Design Patterns & Principles

### Per-DBMS dispatch via `SqlQueryFormatter`
Never branch on `SqlServerType` in business logic. Instead, add a virtual method to `SqlQueryFormatter` and override it per DBMS in `QueryFormatter/{SqlServerFormatter,PostgreSqlFormatter,SQLiteFormatter}.hpp`. Canonical example: `PostgreSqlFormatter::QueryLastInsertId()` returns `"SELECT lastval();"` while the SQL Server override returns `"SELECT @@IDENTITY;"`.

### `SqlDataBinder<T>` specialization for new types

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LASTRADA-Software/Lightweight](https://github.com/LASTRADA-Software/Lightweight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
