---
trigger: always_on
description: Handles connection management including:
---

# AGENTS.md

This file provides guidance for AI coding agents working on the Spirit codebase.

## Project Overview

Spirit is an **online schema change tool for MySQL 8.0+**, reimplementing [gh-ost](https://github.com/github/gh-ost). It applies `ALTER TABLE` statements to large tables without blocking reads or writes by creating a shadow copy, streaming binlog changes, and performing an atomic cutover via `RENAME TABLE`.

Spirit is designed for **speed** — it is multi-threaded in both row-copying and binlog-applying phases. The internal goal is to migrate a 10 TiB table in under 5 days. It has been demonstrated on a real 10 TiB table in ~65 hours.

**Key tradeoffs vs gh-ost:**
- Only supports MySQL 8.0+
- Does not support keeping read replicas within <10s lag
- Targets AWS Aurora environments (InnoDB only, no read-replica fidelity)

## Build & Run

```bash
# Build the spirit binary
cd cmd/spirit && go build

# Run a schema change
./spirit migrate --host=<host> --username=<user> --password=<pass> --database=<db> --table=<table> --alter="<alter statement>"

# Other subcommands
./spirit move --help
./spirit lint --help
./spirit diff --help
./spirit fmt --help
```

Spirit uses [Kong](https://github.com/alecthomas/kong) for CLI argument parsing with subcommands. The CLI structs are defined in `pkg/migration/`, `pkg/move/`, and `pkg/lint/` respectively.

## Requirements

- **Go 1.26+**
- **MySQL 8.0+** for running tests and performing schema changes
- **golangci-lint v2** for linting

## Testing

Tests require a running MySQL server. Provide the DSN via environment variable:

```bash
MYSQL_DSN="root:mypassword@tcp(127.0.0.1:3306)/test" go test -v ./...
```

If `MYSQL_DSN` is not set, it defaults to `spirit:spirit@tcp(127.0.0.1:3306)/test`.

### Running tests with Docker

```bash
cd compose/
docker compose down --volumes && docker compose up -f compose.yml -f 8.0.28.yml
docker compose up mysql test --abort-on-container-exit
```

### Test utilities

The `pkg/testutils/` package provides helpers used across all test files:

- `DSN()` / `DSNForDatabase(dbName)` — returns the MySQL DSN from the environment or default
- `NewTestTable(t, name, createSQL)` — creates a test table with automatic cleanup (see below)
- `CreateUniqueTestDatabase(t)` — creates a unique temporary database with automatic cleanup via `t.Cleanup()`
- `RunSQL(t, stmt)` / `RunSQLInDatabase(t, dbName, stmt)` — execute SQL against the test MySQL

#### `NewTestTable` — preferred way to create test tables

`NewTestTable` handles the full lifecycle of a test table: drops any pre-existing table and Spirit artifacts (`_new`, `_old`, `_chkpnt`), runs the CREATE TABLE, provides a `*sql.DB` connection for verification queries, and registers `t.Cleanup()` to drop everything when the test finishes.

```go
tt := testutils.NewTestTable(t, "mytable",
    `CREATE TABLE mytable (
        id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(255) NOT NULL
    )`)

// Seed with ~1000 rows using INSERT...SELECT doubling
tt.SeedRows(t, "INSERT INTO mytable (name) SELECT 'a'", 1000)

// Use tt.DB for verification queries after migration
var count int
tt.DB.QueryRowContext(t.Context(), "SELECT COUNT(*) FROM mytable").Scan(&count)
```

**`SeedRows` API:** The `insertSelectSQL` argument is an `INSERT INTO ... SELECT` statement **without a FROM clause**. `SeedRows` appends `FROM dual` for the initial insert, then `FROM <table>` for each doubling iteration until the target row count is reached. This design means the same SQL expression works for both the seed and the doubling, and SQL functions like `RANDOM_BYTES()` or `UUID()` can be used naturally.

```go
// Simple seeding — produces ~4096 identical rows (different auto-increment IDs)
tt.SeedRows(t, "INSERT INTO mytable (name, val) SELECT 'seed', 1", 4096)

// With SQL functions — each row gets unique random data
tt.SeedRows(t, "INSERT INTO mytable (pad) SELECT RANDOM_BYTES(1024)", 100000)
```

**When NOT to use `SeedRows`:**
- Tables with composite PKs where you need unique key pairs — use a loop with `RunSQL`
- Rows with specific distinct values needed for the test logic (e.g., inserting specific data that will violate a constraint)

#### `NewTestRunner` — preferred way to create migration runners (migration package only)

`NewTestRunner` is defined in `pkg/migration/helpers_test.go` (only available within the `migration` package tests). It eliminates the repeated `mysql.ParseDSN` / `NewRunner(&Migration{...})` boilerplate:

```go
// Simple migration
m := NewTestRunner(t, "mytable", "ENGINE=InnoDB")
require.NoError(t, m.Run(t.Context()))
assert.NoError(t, m.Close())

// With options
m := NewTestRunner(t, "mytable", "ADD INDEX idx_a (a)",
    WithThreads(1),
    WithTargetChunkTime(100*time.Millisecond),
    WithBuffered(false), // opt out of the default buffered copier
)
```

For tests that use full SQL statements (e.g., `ALTER TABLE ... ADD KEY ... SECONDARY_ENGINE_ATTRIBUTE=...`), use `NewTestRunnerFromStatement`:

```go
m := NewTestRunnerFromStatement(t, "ALTER TABLE mytable ADD COLUMN c INT", WithThreads(1))
require.NoError(t, m.Run(t.Context()))
assert.NoError(t, m.Close())
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [block/spirit](https://github.com/block/spirit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
