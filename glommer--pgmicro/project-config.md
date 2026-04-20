---
trigger: always_on
description: In-process PostgreSQL reimplementation backed by Turso's SQLite-compatible engine.
---

# pgmicro Agent Guidelines

In-process PostgreSQL reimplementation backed by Turso's SQLite-compatible engine.

## Quick Reference

```bash
cargo build                                      # build everything
cargo build -p pgmicro                           # build pgmicro binary only
cargo test -p pgmicro                            # pgmicro integration tests
cargo test -p turso_parser_pg                    # PG parser/translator tests
cargo test --test integration -- postgres        # PG integration tests (catalog, dialect, table)
cargo fmt                                        # format (required)
cargo clippy --workspace --all-features --all-targets -- --deny=warnings  # lint

# Run the pgmicro REPL
cargo run -p pgmicro -- :memory:

# Run with PG wire protocol server
cargo run -p pgmicro -- :memory: --server 127.0.0.1:5432

# Then connect with psql
psql -h 127.0.0.1 -p 5432 -U turso -d main
```

## Architecture

```
  PostgreSQL SQL                          SQLite SQL
       |                                      |
       v                                      v
  libpg_query (C FFI)                   Turso Parser
       | protobuf parse tree                  | Turso AST
       v                                      |
  PostgreSQLTranslator  ----> Turso AST ----->|
  (parser_pg/src/translator.rs)               v
                                     Turso Compiler (core/translate/)
                                              | VDBE bytecode
                                              v
                                     Bytecode Engine (core/vdbe/)
                                              |
                                              v
                                     SQLite Storage (B-tree, WAL)
```

Key: PG SQL is parsed by PostgreSQL's own parser (via libpg_query), then translated
**directly to Turso's AST** — never re-serialized as SQLite text. From the AST onward,
the entire Turso compilation and execution pipeline is shared.

## Structure

```
limbo/
├── parser_pg/              # PG parser + translator crate
│   ├── src/
│   │   ├── lib.rs          # Public API: parse(), split_statements()
│   │   └── translator.rs   # PG protobuf → Turso AST (5.5k LOC, the heart of pgmicro)
│   └── tests/              # Parse validity and error tests
├── pgmicro/                # Standalone binary crate
│   ├── src/main.rs         # REPL, meta-commands (\dt, \d, \l), --server flag
│   ├── build.rs            # Syntax highlighting compilation
│   └── tests/pgmicro.rs    # Binary integration tests (stdin/stdout)
├── core/
│   ├── pg_catalog.rs       # PG catalog virtual tables (3.6k LOC)
│   ├── functions/postgres.rs # PG system functions (pg_get_userbyid, format_type, etc.)
│   ├── connection.rs       # Dialect dispatch: parse_postgresql_sql(), try_prepare_pg()
│   └── lib.rs              # SqlDialect enum, DatabaseOpts::enable_postgres
├── cli/pg_server.rs        # PG wire protocol server (pgwire crate, 800 LOC)
├── npm/pgmicro/            # npm package (pg-micro)
├── tests/integration/postgres/
│   ├── catalog.rs          # PG catalog virtual table tests
│   ├── dialect.rs          # Dialect switching + PG query tests
│   └── table.rs            # Data read-through tests
└── bindings/javascript/    # NAPI bindings (shared with tursodb, PG-aware)
```

## Where to Look

| Task | Location | Notes |
|------|----------|-------|
| Translate new PG syntax | `parser_pg/src/translator.rs` | Map pg_query protobuf nodes to `turso_parser::ast` |
| Add PG catalog table | `core/pg_catalog.rs` | Implement `InternalVirtualTable` trait |
| Add PG system function | `core/functions/postgres.rs` | Register in `core/function.rs` |
| Wire protocol issues | `cli/pg_server.rs` | `pgwire` crate, type encoding/decoding |
| Dialect switching | `core/connection.rs` | `parse_postgresql_sql()`, `try_prepare_pg()` |
| Meta-commands (\dt, \d) | `pgmicro/src/main.rs` | Inline in the REPL loop |
| Schema support | `core/connection.rs` | CREATE/DROP SCHEMA → ATTACH/DETACH |
| PG type mapping | `core/pg_catalog.rs` | `sqlite_type_to_pg_oid()`, type OID tables |
| Wire type encoding | `cli/pg_server.rs` | `pg_bytes_to_value()`, `encode_value()` |
| Test PG features | `tests/integration/postgres/` | Rust API tests (dialect, catalog, table) |
| Test pgmicro binary | `pgmicro/tests/pgmicro.rs` | Spawns binary, pipes stdin |

## The Translator: How PG SQL Becomes Bytecode

The translator (`parser_pg/src/translator.rs`) is the most important file. It converts
`pg_query::ParseResult` (protobuf from PostgreSQL's actual C parser) into `turso_parser::ast::Stmt`.

### Flow

1. `turso_parser_pg::parse(sql)` → calls `pg_query::parse()` (libpg_query C FFI) → protobuf tree
2. `PostgreSQLTranslator::translate(&parse_result)` → dispatches on node type:
   - `SelectStmt` → `translate_select()` (CTEs, UNION, ORDER BY, LIMIT)
   - `InsertStmt` → `translate_insert()` (RETURNING, ON CONFLICT)
   - `CreateStmt` → `translate_create_table()` (column defs, constraints, SERIAL)
   - `UpdateStmt`, `DeleteStmt`, `DropStmt`, `AlterTableStmt`, etc.
3. `translate_expr()` is the central expression dispatcher — handles:
   - `ColumnRef` → `ast::Expr::Id` / `ast::Expr::Qualified`
   - `A_Const` → `ast::Expr::Literal`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [glommer/pgmicro](https://github.com/glommer/pgmicro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
