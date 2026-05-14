---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Building

```bash
# Production build (outputs to dist/)
npm run build

# Development build (outputs to demo_vault/.obsidian/plugins/decks)
npm run build:dev

# Clean build artifacts
npm run clean

# Full release build (clean + build)
npm run build:release
```

### Testing

```bash
# Run all unit tests (excludes integration tests)
npm test
# or
npm run test:unit

# Run only integration tests (uses real SQL.js database)
npm run test:integration

# Run all tests (unit + integration)
npm run test:all
```

**Note:** Integration tests run serially (`maxWorkers: 1`) to avoid database conflicts and have longer timeouts (30s).

### Code Quality

```bash
# Lint TypeScript and Svelte files
npm run lint

# Auto-fix linting issues
npm run lint:fix

# Type check Svelte components
npm run check
```

### Code Rules

- Avoid using "any" or "unknown" as types, use available types or create if necessary.
- Avoid casting if possible but when it's needed use proper defined types instead of defining the casting type on the spot.
- Css classes need to have the prefix decks-
- No excessive unnecessary comments. No comments that mention any prompt or plan. Minimal comments explaining functionality allowed.

### Obsidian Plugin Store Linting Rules

- **Logging**:
  - Prefer using the `Logger` class methods (`logger.debug()`, `logger.error()`) for unified logging across the plugin
  - When Logger is not available, only use `console.debug()`, `console.warn()`, and `console.error()`
  - Never use `console.log()` or `console.info()` - they are not allowed by Obsidian plugin store linter
- **Async Modal methods**: Modal and ItemView lifecycle methods (`onOpen()`, `onClose()`) must NOT be async. Wrap async operations in separate methods called from lifecycle methods with `.catch(console.error)`.
- **DOM manipulation**: Never use `innerHTML` or `outerHTML`. Use Obsidian's `createEl()`, `createDiv()`, `appendText()`, etc.
- **Style properties**: Never set `element.style.*` directly. Use CSS classes with `decks-` prefix and `addClass()`/`removeClass()`, or use `element.setCssProps()` for dynamic positioning.
- **UI headings**: Use `new Setting(containerEl).setName("...").setHeading()` instead of `containerEl.createEl("h2"|"h3", ...)`.
- **Deprecated methods**: Use `substring()` or `slice()` instead of deprecated `substr()`.
- **eslint-disable**: Never disable `@typescript-eslint/no-explicit-any` rule. Fix the underlying type issue instead.
- **Sentence case**: All UI text must use sentence case, not title case (e.g., "Review sessions" not "Review Sessions").
- **Node.js imports**: In test files, use `node:` protocol (e.g., `import from "node:fs"` not `import from "fs"`).

### Releasing

```bash
# Create a git tag (triggers automated GitHub Actions release)
git tag v1.2.3 && git push origin v1.2.3
```

GitHub Actions will automatically:

- Run all tests
- Build the plugin
- Generate release notes from `docs/PROGRESS.md`
- Create GitHub release with all dist/ files

## Architecture Overview

### Layered Architecture

The codebase follows a strict layered architecture with clear separation of concerns:

```
UI Layer (Svelte Components)
    ↓
Service Layer (TypeScript Business Logic)
    ↓
Database Layer (SQL.js with Main/Worker Variants)
    ↓
Storage Layer (Obsidian Vault)
```

### Key Architectural Patterns

1. **Factory Pattern**: `DatabaseFactory` creates either `MainDatabaseService` (runs in main thread) or `WorkerDatabaseService` (offloads to Web Worker) based on settings
2. **Abstract Base Class**: `BaseDatabaseService` contains 1,467 lines of shared business logic; concrete implementations only handle SQL execution
3. **Strategy Pattern**: FSRS algorithm encapsulated with profile-based parameters (STANDARD vs INTENSIVE)
4. **Worker Pattern**: Heavy parsing/sync operations can be offloaded to Web Workers for performance
5. **Service Injection**: Services created once in main.ts and injected into components via props (no DI framework)

## Database Architecture

### Dual Execution Modes

The database layer has two implementations that share the same business logic:

- **MainDatabaseService**: Runs SQL.js in main thread, direct database manipulation
- **WorkerDatabaseService**: Communicates with Web Worker via message passing, main thread handles I/O (loading db file, SQL.js assets), worker handles SQL execution

**Key Design Decision**: `BaseDatabaseService` contains ALL business logic (queries, row parsing, batch operations). Concrete implementations only provide:

- `executeSql()` / `querySql()` - execute SQL statements
- `initialize()` / `close()` / `save()` - lifecycle management
- `exportDatabaseToBuffer()` / `syncWithDisk()` - persistence

### Merge-Before-Save Synchronization

**Critical for multi-device sync**: Before saving the database to disk, the code checks if the disk file has been modified by another device. If so, it performs a SQL-based merge:

1. `ATTACH DATABASE` to mount the remote (disk) database
2. Review data: `INSERT OR IGNORE` (preserve all review history from both devices)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dscherdi/decks](https://github.com/dscherdi/decks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
