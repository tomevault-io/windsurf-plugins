---
trigger: always_on
description: SQLKit is a **Tauri-based cross-platform SQL database GUI client** with an AI-powered interface. The architecture splits into two distinct parts:
---

# SQLKit Copilot Instructions

## Project Overview

SQLKit is a **Tauri-based cross-platform SQL database GUI client** with an AI-powered interface. The architecture splits into two distinct parts:

- **Frontend**: Vue 3 + TypeScript + UnoCSS (shadcn-vue components)
- **Backend**: Rust with async database adapters for PostgreSQL, MySQL, SQL Server, and SQLite

## Architecture

### Rust Backend (`src-tauri/`)

The core abstraction is the **`DatabaseAdapter` trait** ([src-tauri/src/database/adapter.rs](src-tauri/src/database/adapter.rs)) which provides a unified interface for all database operations:

```rust
#[async_trait]
pub trait DatabaseAdapter: Send + Sync {
    type Pool: ConnectionPool;
    async fn connect(&mut self) -> DbResult<()>;
    async fn execute_query(&self, query: &str) -> DbResult<QueryResult>;
    async fn list_databases(&self) -> DbResult<Vec<DatabaseSchema>>;
    // ... metadata retrieval methods
}
```

Each database has its own adapter ([postgres.rs](src-tauri/src/database/postgres.rs), [mysql.rs](src-tauri/src/database/mysql.rs), [sqlserver.rs](src-tauri/src/database/sqlserver.rs), [sqlite.rs](src-tauri/src/database/sqlite.rs)) implementing this trait with database-specific connection pooling.

**Connection Management Pattern**:

- `ConnectionManager<P>` ([manager.rs](src-tauri/src/database/manager.rs)) wraps pools to add health checks, metadata tracking, and lifecycle management
- Each adapter has its own `*Pool` struct implementing the `ConnectionPool` trait ([pool.rs](src-tauri/src/database/pool.rs))
- Pools use database-specific clients: `deadpool-postgres`, `mysql_async`, `tiberius`, `rusqlite`

**Configuration Builders**: Use fluent builder pattern:

```rust
ConnectionConfig::new(DatabaseType::PostgreSQL, "localhost", 5432, "user")
    .with_database("mydb")
    .with_password("pass")
    .with_ssl_mode(SslMode::Require)
```

### Frontend (`src/`)

- **Layout Structure**: `AppLayout.vue` → `AppHeader.vue` + `AppSidebar.vue` + slot for main content
- **UI Components**: shadcn-vue based components in `src/components/ui/` (Button, Card, Dialog, Input, Label, Table)
- **Monaco Editor Integration**: `useMonacoEditor.ts` composable configures SQL syntax highlighting and autocomplete with custom worker setup for Vite
- **Theme System**: `useTheme.ts` composable + CSS variables in `assets/index.css` for light/dark modes

**Styling**: UnoCSS with Wind4 preset + shadcn preset ([uno.config.ts](uno.config.ts)). Uses utility-first approach with Tailwind-like syntax.

## Development Workflows

### Running the App

```bash
npm install                # Install frontend deps
npm run tauri dev          # Run in dev mode (starts Vite + Tauri)
```

Vite runs on **port 1420** (fixed, will fail if unavailable). HMR runs on 1421.

### Testing

**Frontend Tests**: Jest with ts-jest

```bash
npm test                   # Run with coverage
npm run test:ci            # CI mode
```

**Rust Tests**: Standard Cargo tests + integration tests in `src-tauri/tests/`

```bash
cd src-tauri
cargo test                 # Unit tests in *_tests.rs and integration tests
cargo test --test postgres_integration  # Specific integration test
```

Integration tests require actual database instances (see [BUILD.md](BUILD.md) for setup).

### Linting & Formatting

- **Frontend**: `@antfu/eslint-config` with auto-fix on pre-commit
- **Rust**: Standard `rustfmt` and `clippy`

```bash
npm run lint:fix           # Fix frontend issues
cargo fmt                  # Format Rust code
cargo clippy               # Rust linter
```

## Conventions & Patterns

### Rust Error Handling

All database operations return `DbResult<T>` (type alias for `Result<T, DbError>`). The `DbError` enum ([error.rs](src-tauri/src/database/error.rs)) has specific variants for Connection, Authentication, Query, Timeout, Pool errors.

### Async Patterns

- All adapter methods use `#[async_trait]` for trait async support
- Use Tokio runtime (`tokio = { version = "1", features = ["full"] }`)
- Connection pools return `Arc<Connection>` for shared ownership
- **State synchronization**: Use `tokio::sync::Mutex` for state accessed across Tauri commands (not `std::sync::Mutex` - it's not Send across await points)
- In tests: Use `blocking_lock()` for tokio::Mutex when in synchronous test setup functions

### Type Conversions

Each adapter implements database-specific type mapping to `QueryValue` enum:

- PostgreSQL: Handles arrays, JSON/JSONB, custom types
- MySQL: JSON columns, binary data, various numeric types
- SQL Server: XML, UNIQUEIDENTIFIER, DATETIME2, hierarchyid
- SQLite: Limited type system (NULL, INTEGER, REAL, TEXT, BLOB)

### Vue Component Structure

Use `<script setup lang="ts">` syntax with Composition API. Import UI components from relative paths (aliased with `@/` for src).

### Tauri Commands

Commands are defined with `#[tauri::command]` and registered in [lib.rs](src-tauri/src/lib.rs) via `generate_handler![]`.

**Implemented Commands** (15 total across 4 modules):

- **Server Management** ([server.rs](src-tauri/src/commands/server.rs)): `save_server`, `list_servers`, `delete_server`, `test_connection`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [geek-fun/sqlkit](https://github.com/geek-fun/sqlkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
