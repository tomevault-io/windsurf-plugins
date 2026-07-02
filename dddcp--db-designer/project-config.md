---
trigger: always_on
description: **Generated:** 2026-06-22
---

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-06-22
**Commit:** 2c93699
**Branch:** main

## OVERVIEW

Tauri v2 desktop app for visually designing database table structures and generating SQL. Supports AI-powered table design, version management, remote DB comparison/sync (MySQL/PostgreSQL/Oracle), SQL export, Git integration, and local SQLite persistence.

**Stack:** React 18 + TypeScript + Ant Design 5 (frontend) | Rust + SQLite via rusqlite (backend) | Vite 7 + Tauri 2

## STRUCTURE

```
├── src/                        # React frontend
│   ├── components/
│   │   ├── main/               # Project list page (1 file)
│   │   ├── proj-detail/        # Core design workspace (15 files) ← SEE AGENTS.md
│   │   └── setting/            # Settings tabs (6 files)
│   ├── i18n/                   # i18next (zh-CN, en-US)
│   ├── store/                  # theme-context.tsx (only global state)
│   ├── types/index.ts          # Shared TS types (mirrors models.rs)
│   └── data-types.ts           # 19 built-in + custom DB column types
├── src-tauri/src/              # Rust backend ← SEE AGENTS.md
│   ├── lib.rs                  # Plugin + command registry (58 commands)
│   ├── models.rs               # Shared Rust structs (mirrors types/index.ts)
│   ├── db.rs                   # SQLite init + migrations
│   ├── dialect.rs              # DatabaseDialect + DatabaseConnector traits
│   ├── {project,table,setting,git,version,sync,routine,db_connection,ai_review,ai_sql}.rs  # Command modules (flat, no subfolder)
│   ├── services/               # Business logic layer (7 services)
│   └── storage/                # Trait definitions + sqlite/ implementations
├── scripts/bump-version.ps1    # Sync version across 3 config files
└── openspec/                   # OpenSpec workflow (changes + specs)
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Add a new Tauri command | `src-tauri/src/<module>.rs` + `lib.rs` + `services/` + `storage/sqlite/` | Must register in `generate_handler![]` |
| Add a new DB dialect | `src-tauri/src/dialect.rs` | Implement both `DatabaseDialect` + `DatabaseConnector`, add to factories |
| Add a new frontend tab | `src/components/proj-detail/<name>-tab.tsx` | Import into `index.tsx`, add to Tabs component |
| Add a new settings section | `src/components/setting/<name>-tab.tsx` | Import into `setting.tsx` |
| Add a new shared type | `src/types/index.ts` (TS) + `src-tauri/src/models.rs` (Rust) | Must keep in sync |
| Add SQLite storage | `src-tauri/src/storage/mod.rs` (trait) + `storage/sqlite/<name>_store.rs` (impl) | Follow existing Store trait pattern |
| Add a new service | `src-tauri/src/services/<name>_service.rs` + register in `mod.rs` | Services take `Box<dyn XxxStore>` |
| Change DB schema | `src-tauri/src/db.rs::init_database()` | Add `CREATE TABLE` or `ALTER TABLE` migration |
| Change i18n strings | `src/i18n/locales/zh-CN.json` + `en-US.json` + `backend-messages.ts` | Backend error messages in `backend-messages.ts` |
| Debug IPC issues | Browser DevTools → `invoke('command_name', {...})` | Frontend camelCase, backend snake_case |

## CODE MAP

### Frontend Key Symbols

| Symbol | Location | Role |
|--------|----------|------|
| `App` | `src/App.tsx` | Root: ThemeProvider → ConfigProvider → Router |
| `Main` | `src/components/main/main.tsx` | Route `/` — project list, Git sync, auto-update |
| `ProjectDetail` | `src/components/proj-detail/index.tsx` | Route `/project/:id` — core workspace (1235 lines) |
| `Setting` | `src/components/setting/setting.tsx` | Route `/setting` — tabs for basic/AI/DB/Git/data-type |
| `useTheme` | `src/store/theme-context.tsx` | Only global context: dark mode via localStorage |
| `BUILT_IN_DATA_TYPES` | `src/data-types.ts` | 19 built-in column types + custom type CRUD |
| `i18n` | `src/i18n/index.ts` | i18next init: localStorage → navigator → zh-CN fallback |

### Backend Key Symbols

| Symbol | Location | Role |
|--------|----------|------|
| `run()` | `lib.rs:24` | Tauri Builder entry — registers 5 plugins + 58 commands |
| `DatabaseDialect` trait | `dialect.rs` | SQL generation methods (CREATE, ALTER, DROP, comments, type maps) |
| `DatabaseConnector` trait | `dialect.rs` | Remote DB connection + table/routine introspection |
| `get_dialect()` / `get_connector()` | `dialect.rs` | Factory functions by `db_type` string |
| `init_database` | `db.rs` | SQLite schema creation + 3 field migrations |
| `init_db()` | `db.rs` | SQLite connection (respects `DB_DESIGNER_DATA_PATH` env) |
| Storage traits | `storage/mod.rs` | `ProjectStore`, `TableStore`, `VersionStore`, etc. |
| SQLite impls | `storage/sqlite/` | Concrete `Box<dyn XxxStore>` implementations |
| Service layer | `services/` | Business logic coordinating dialect + storage |

## CONVENTIONS

- **No ESLint/Prettier** — verification via `npx tsc --noEmit` + `cargo check` only
- **No test suite** — no Jest, Vitest, or `#[cfg(test)]` exists
- **ESM only** — `package.json` has `"type": "module"`
- **Tauri IPC naming** — backend `snake_case` params, frontend `camelCase` params (auto-converted)
- **Error handling** — all Tauri commands return `Result<T, String>` with `.map_err(|e| format!(...))`
- **Backend layering enforcement** — `command → service → storage`; never put SQL or business logic in command modules

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dddcp/db-designer](https://github.com/dddcp/db-designer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
