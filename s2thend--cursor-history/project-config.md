---
trigger: always_on
description: CLI tool and library to browse, search, and export Cursor AI chat history. Built with TypeScript, commander, pluggable SQLite drivers (better-sqlite3 or node:sqlite), and picocolors.
---

# cursor-history Development Guidelines

## Overview

CLI tool and library to browse, search, and export Cursor AI chat history. Built with TypeScript, commander, pluggable SQLite drivers (better-sqlite3 or node:sqlite), and picocolors.

**Dual Interface:**
- **CLI**: Command-line tool for interactive use (`cursor-history list`, `cursor-history show 1` or `show <composer-id>`)
- **Library**: Programmatic API for integration (`import { listSessions } from 'cursor-history'`)

## Quick Reference

```bash
# Build and run
npm run build && node dist/cli/index.js list

# Development
npm run dev          # Watch mode
npm test             # Run tests
npm run lint         # Lint code
npm run typecheck    # Type check
```

## Architecture

### Data Flow

1. **Workspace storage** (`workspaceStorage/*/state.vscdb`)
   - Contains session metadata with correct workspace paths
   - User messages stored in `ItemTable` under `composer.composerData`

2. **Global storage** (`globalStorage/state.vscdb`)
   - Contains full AI responses in `cursorDiskKV` table
   - Keys: `composerData:<id>` (metadata), `bubbleId:<composerId>:<bubbleId>` (messages)

3. **Bubble extraction priority** (for assistant messages):
   - Mark errors early: `toolFormerData.additionalData.status === 'error'` but continue extraction
   - `toolFormerData.result` → check for diff blocks (write/edit operations)
   - `toolFormerData.name` + `status: "completed"` → standard tool calls with params
   - `text` field → natural language explanation (check for JSON diff if starts with `{`)
   - `codeBlocks[].content` → code/mermaid artifacts (COMBINED with text, wrapped in ```lang fences)
   - `thinking.text` → reasoning blocks (marked as `[Thinking]`)
   - **Last resort**: Recursive walk through all fields to find longest string with markdown features (catches error messages)
   - If marked as error, prefix result with `[Error]` marker
   - All extractions include timestamps for display

### Project Structure

```
src/
├── cli/
│   ├── commands/          # list, show, search, export, migrate, migrate-session
│   ├── formatters/        # table.ts (terminal), json.ts
│   ├── errors.ts          # CLI-specific errors (CliError, SessionNotFoundError)
│   └── index.ts           # CLI entry, global options
├── core/
│   ├── database/          # Pluggable SQLite driver abstraction
│   │   ├── drivers/       # better-sqlite3.ts, node-sqlite.ts adapters
│   │   ├── types.ts       # Database, Statement, DatabaseDriver interfaces
│   │   ├── registry.ts    # DriverRegistry singleton (auto-select, manual set)
│   │   ├── errors.ts      # NoDriverAvailableError, DriverNotAvailableError
│   │   ├── debug.ts       # Debug logging utility
│   │   └── index.ts       # Public API (openDatabase, setDriver, getActiveDriver)
│   ├── storage.ts         # findWorkspaces, listSessions, getSession, extractBubbleText
│   ├── migrate.ts         # migrateSession, migrateWorkspace, copyBubbleDataInGlobalStorage
│   ├── backup.ts          # createBackup, restoreBackup, openBackupDatabase
│   ├── parser.ts          # parseChatData, exportToMarkdown, exportToJson
│   └── types.ts           # ChatSession, Message, Workspace, ToolCall, MigrationMode, etc.
└── lib/
    ├── index.ts           # Library entry point (listSessions, getSession, searchSessions, export*, migrate*, setDriver, getActiveDriver)
    ├── types.ts           # Public library types (Session, Message, SearchResult, MigrateSessionConfig, SqliteDriverName, etc.)
    ├── config.ts          # Configuration validation and merging (including sqliteDriver)
    ├── errors.ts          # Library errors (DatabaseLockedError, SessionNotFoundError, WorkspaceNotFoundError, etc.)
    ├── utils.ts           # Utility functions (getDefaultDataPath)
    └── platform.ts        # getCursorDataPath, expandPath, contractPath, normalizePath, pathsEqual
```

### Architecture: Shared Core

Both CLI and Library share the same core logic:

```
┌─────────────────────────────────────────────────────────────┐
│                      src/core/                              │
│  storage.ts (DB queries)  +  parser.ts (data parsing)       │
│                    ↑               ↑                        │
└────────────────────┼───────────────┼────────────────────────┘
                     │               │
        ┌────────────┴───────────────┴────────────┐
        │                                         │
        ▼                                         ▼
┌───────────────────┐                 ┌───────────────────────┐
│   src/cli/        │                 │   src/lib/            │
│   commands/       │                 │   index.ts            │
│   (CLI interface) │                 │   (Library API)       │
└───────────────────┘                 └───────────────────────┘
```

**Both share:**
- `src/core/storage.ts` - `listSessions()`, `getSession()`, `searchSessions()`, `findWorkspaceForSession()`, `findWorkspaceByPath()`
- `src/core/migrate.ts` - `migrateSession()`, `migrateWorkspace()`, `copyBubbleDataInGlobalStorage()`
- `src/core/parser.ts` - `exportToJson()`, `exportToMarkdown()`

**The library adds:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [S2thend/cursor-history](https://github.com/S2thend/cursor-history) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
