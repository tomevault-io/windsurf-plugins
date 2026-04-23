---
trigger: always_on
description: This is a Go implementation of SQLsmith - a SQL query generator/fuzzer for testing database systems. The project supports multiple SQLite-compatible database flavors with different feature sets and compatibility constraints.
---

# SQLfuse AI Coding Instructions

## Project Overview
This is a Go implementation of SQLsmith - a SQL query generator/fuzzer for testing database systems. The project supports multiple SQLite-compatible database flavors with different feature sets and compatibility constraints.

## Architecture & Key Components

### Database Flavors
The project supports multiple database flavors, each with different SQL feature sets:
- **Turso LibSQL**: SQLite-compatible with some restrictions (see [COMPAT.md](https://github.com/tursodatabase/turso/blob/main/COMPAT.md))
- **go-sqlite3**: Full SQLite3 support via `github.com/mattn/go-sqlite3`
- **DuckDB**: Analytical database with extensive SQL features
- **Default/Unknown**: Conservative mode (Turso-compatible subset for safety)

### Flavor Configuration
- Flavor configs are in `internal/generators/dialects/` directory
- Each flavor implements the `stmts.FlavorConfig` interface:
  - `Name() string`: Returns flavor identifier ("turso", "go-sqlite3", etc.)
  - `SupportsFeature(feature string) bool`: Feature capability checking
  - `ValidateSQL(sql string) error`: Optional SQL validation
- Statement generators use flavor configs to generate compatible SQL
- **Key principle**: When generating SQL statements or pragmas, always check the flavor to ensure compatibility

### Database Integration
- **Turso LibSQL**: SQLite-compatible via `github.com/tursodatabase/go-libsql`
- **go-sqlite3**: Full SQLite3 via `github.com/mattn/go-sqlite3`
- **SQL parsing**: Uses ANTLR4-based SQLite parser (`github.com/libsql/sqlite-antlr4-parser`)
- **Connection pattern**: Standard `database/sql` interface with flavor-specific drivers

### Project Structure
```
cmd/executors/               # Database executor implementations
├── turso_embedded/         # Turso LibSQL executor
├── go_sqlite3_embedded/    # go-sqlite3 executor
└── duckdb_embedded/        # DuckDB executor
internal/
├── generators/             # SQL statement generators
│   ├── dialects/          # Flavor-specific configurations
│   │   ├── turso.go       # Turso LibSQL flavor config
│   │   ├── go_sqlite3.go  # go-sqlite3 flavor config
│   │   └── duckdb.go      # DuckDB flavor config
│   ├── go_sqlite3.go      # go-sqlite3 generator
│   ├── turso.go           # Turso generator (base)
│   └── duckdb.go          # DuckDB generator
└── stmts/stmts/           # Statement generation logic
    ├── pragma.go          # Flavor-aware PRAGMA generation
    └── ...                # Other statement types
```

## SQL Statement Generation with Flavors

### Flavor-Aware Statement Generation
When generating SQL statements (especially PRAGMA, DDL, or advanced SQL features):

1. **Check the flavor first**: Use `flavor.Name()` to identify the target database
2. **Use flavor feature checks**: Call `flavor.SupportsFeature(feature)` for conditional generation
3. **Generate appropriate SQL**: Adjust syntax, features, and parameters based on flavor

### PRAGMA Statement Generation
PRAGMA generation is flavor-aware (see `internal/stmts/stmts/pragma.go`):

- **Turso/Default flavor** (18 pragmas):
  - Limited to Turso-compatible pragmas ([reference](https://github.com/tursodatabase/turso/blob/main/COMPAT.md#pragma))
  - `journal_mode`: Only WAL supported
  - `synchronous`: Only OFF and FULL
  - `table_info`: No table name parameter
  - `wal_checkpoint`: No checkpoint mode parameter

- **go-sqlite3 flavor** (47 pragmas):
  - Full SQLite3 pragma support ([reference](https://sqlite.org/pragma.html))
  - All journal modes: DELETE, TRUNCATE, PERSIST, MEMORY, WAL, OFF
  - All synchronous modes: OFF, NORMAL, FULL, EXTRA
  - Extended pragmas: `auto_vacuum`, `foreign_keys`, `busy_timeout`, `mmap_size`, `threads`, etc.
  - Parameterized pragmas: `table_info(table_name)`, `wal_checkpoint(mode)`, etc.

### Adding Flavor Support to Statements
When implementing new statement generators:
```go
func genStatementWithFlavor(lcg *common.LCG, flavor FlavorConfig) Stmt {
    if flavor.Name() == "go-sqlite3" {
        // Generate with full SQLite3 features
    } else {
        // Generate conservative/Turso-compatible SQL
    }
}
```

## Development Patterns

### Database Connection Pattern
Always use the appropriate connection pattern for each database flavor:

**Turso LibSQL:**
```go
dbName := "file:./local.db"
db, err := sql.Open("libsql", dbName)
// Always defer db.Close() and handle connection errors with os.Exit(1)
```

**go-sqlite3:**
```go
dbName := "./local.db"  // or "file:./local.db"
db, err := sql.Open("sqlite3", dbName)
// Always defer db.Close() and handle connection errors with os.Exit(1)
```

### Query Execution Pattern
Follow the established pattern across all executors:
- Use `db.Query()` for SELECT statements
- Always defer `rows.Close()`
- Handle `rows.Err()` after iteration
- Print errors to `os.Stderr` and exit with status 1 for fatal errors

### Error Handling Convention
- Database connection errors: `fmt.Fprintf(os.Stderr, ...)` + `os.Exit(1)`
- Query execution errors: Print to stderr and exit
- Row scanning errors: Print to stdout and return (non-fatal)

## Build & Development Workflow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyw0ng95/sqlfuse](https://github.com/cyw0ng95/sqlfuse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
