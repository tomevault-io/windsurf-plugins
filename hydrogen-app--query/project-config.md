---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Query is a desktop SQL database client built with Tauri 2.0, React 19, and TypeScript. It provides a Monaco-based SQL editor with IntelliSense, schema browsing, query history, and saved queries. The application uses PostgreSQL for remote databases and SQLite for local data storage (history, saved queries). Passwords are stored securely in the OS keychain.

**Tech Stack:**
- **Frontend**: React 19.1 + TypeScript 5.8 + Vite 7 + Tailwind CSS 4
- **Backend**: Tauri 2.0 + Rust (async with Tokio)
- **Database**: SQLx 0.8.6 (PostgreSQL, SQLite, MySQL support)
- **Editor**: Monaco Editor 4.7 with SQL IntelliSense and optional Vim mode
- **Data Tables**: TanStack Table 8.21 with sorting, filtering, virtual scrolling

## Development Commands

### Running the Application
```bash
# Development mode (hot reload for both frontend and backend)
bun run tauri dev

# Or with bun (as configured in tauri.conf.json)
bun run dev  # Frontend only
bun run tauri dev  # Full Tauri app

# Build production version
bun run build      # Build frontend
cargo build        # Build Rust backend
bun run tauri build  # Build complete desktop app
```

### Testing Compilation
```bash
# TypeScript compilation check
bun run build

# Rust compilation check
cd src-tauri && cargo build

# Run Rust tests (if any)
cd src-tauri && cargo test
```

### Frontend Development
```bash
bun run dev      # Start Vite dev server (port 1420)
bun run preview  # Preview production build
```

## Architecture

### Frontend Structure (src/)

The frontend follows a domain-driven organization:

```
src/
├── types/          # Centralized TypeScript types
│   ├── database.ts  # ConnectionConfig, DatabaseSchema, TableInfo, ColumnInfo
│   ├── query.ts     # QueryResult, QueryHistoryEntry, SavedQuery
│   └── index.ts     # Re-exports all types
├── utils/
│   ├── tauri.ts     # Type-safe wrappers for all Tauri commands
│   └── format.ts    # Formatting utilities
├── constants/
│   └── shortcuts.ts # Keyboard shortcuts and default values
└── components/
    ├── editor/      # SqlEditor with Monaco + Vim mode
    ├── results/     # ResultsTableEnhanced (TanStack), ResultsTable (legacy)
    ├── sidebar/     # SchemaExplorer, QueryHistory, SavedQueries
    └── modals/      # CommandPalette (Cmd+K), SaveQueryModal, ProjectSettings
```

**Key Frontend Patterns:**
- All Tauri commands are called through `src/utils/tauri.ts` wrappers (never invoke directly)
- Types are imported from `src/types` (never define inline in components)
- Components are one level deep in domain folders (editor/, results/, sidebar/, modals/)

### Backend Structure (src-tauri/src/)

The Rust backend is organized into domain modules (refactored from a 683-line monolith):

```
src-tauri/src/
├── lib.rs          # Minimal entry point (~50 lines), registers all Tauri commands
├── models/         # Data structures shared across modules
│   ├── connection.rs  # ConnectionConfig
│   ├── query.rs       # QueryResult, QueryHistoryEntry, SavedQuery
│   └── schema.rs      # DatabaseSchema, TableInfo, ColumnInfo
├── commands/       # Tauri command handlers (public API)
│   ├── connection.rs       # test_postgres_connection, execute_query, get_database_schema
│   ├── history.rs          # save_query_to_history, get_query_history, clear_query_history
│   ├── saved_queries.rs    # save_query, get_saved_queries, delete_saved_query, toggle_pin_query
│   └── settings.rs         # Project path, connection storage, keychain wrappers
├── storage/        # Data persistence layer
│   ├── history_db.rs       # SQLite schema and connection for query history
│   ├── saved_queries_db.rs # SQLite schema and connection for saved queries
│   ├── keychain.rs         # OS keychain integration (passwords never touch disk)
│   └── connections.rs      # connections.json file I/O
└── utils/
    └── app_dir.rs  # Project path management, settings persistence
```

**Key Backend Patterns:**
- Each module is focused and small (50-150 lines)
- Commands in `commands/` are public and registered in `lib.rs`
- Storage layer abstracts database/file operations
- Global state: `PROJECT_PATH` mutex in `utils/app_dir.rs` for custom project directories

### Data Storage Architecture

**Local Storage (SQLite):**
- `{project_path}/history.db` - Query execution history (last 20 by default)
- `{project_path}/saved_queries.db` - Named/pinned queries
- `{project_path}/connections.json` - Connection configs (WITHOUT passwords)
- `~/.query/settings.json` - Global app settings (project path preference)

**Secure Storage (OS Keychain):**
- Service: "Query"
- Account: connection name
- Stores: database passwords only

**Project Path Logic:**
- Default: `~/.query/`
- Custom: User-selected directory (set via ProjectSettings modal)
- Changing project path reloads all data from new location

### Frontend-Backend Communication

All communication happens through Tauri commands defined in `src/utils/tauri.ts`:

**Connection Commands:**
- `testPostgresConnection(config)` → Test connection validity
- `getDatabaseSchema(config)` → Fetch tables and columns from information_schema

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hydrogen-app/Query](https://github.com/hydrogen-app/Query) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
