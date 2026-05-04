---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Octopus is a Tauri 2 desktop app (React 19 + Rust) that serves as a dispatch board for managing multiple Claude Code sessions in parallel. It provides a kanban-style UI to track session status, integrated terminal/editor, git operations, and GitHub integration.

## Commands

### Frontend (from repo root)
```bash
npm run dev              # Vite dev server
npm run build            # tsc --noEmit + vite build
npm run test             # vitest unit tests (once)
npm run test:watch       # vitest watch mode
npm run test:integration # vitest integration tests (mockIPC)
npm run test:all         # unit + integration tests
npm run lint             # eslint
npm run lint:fix         # eslint --fix
npm run format           # prettier --write
npm run format:check     # prettier --check
npm run check            # full CI check (lint + format + test:all + typecheck)
```

Run a single test file:
```bash
npx vitest run src/hooks/__tests__/useAsync.test.ts
npx vitest run --config vitest.integration.config.ts src/__tests__/integration/app-navigation.test.tsx
```

### Backend (from repo root, Cargo workspace is configured)
```bash
cargo build          # debug build
cargo build --release
cargo fmt -- --check # format check
cargo clippy -- -D warnings  # lint
cargo test           # run Rust tests
```

### Full app
```bash
npm run tauri dev    # run Tauri app in dev mode
npm run tauri build  # production build
```

## Architecture

### Frontend → Backend Communication
- **Commands**: Frontend calls Rust via `invoke("command_name", { args })` through wrapper functions in `src/lib/tauri.ts`
- **Events**: Backend emits events (`session-state-changed`, `session-output`) that frontend subscribes to via the `useTauriEvent` hook
- **Type mapping**: Rust uses snake_case with serde rename to camelCase; `mapBackendSession()` in tauri.ts handles the conversion

### State Management (Zustand)
Each store in `src/stores/` manages a domain: `sessionStore` (sessions + structured message buffers + streaming state), `uiStore` (panel sizes, persisted to localStorage), `editorStore` (open tabs), `gitStore` (staging/diffs), `fileBrowserStore`, `repoStore`, `themeStore`, `hookStore` (pending permission requests from Claude CLI hooks).

### Rust Backend (`src-tauri/src/`)
- `state.rs`: `AppState` holds `parking_lot::Mutex<Connection>` (SQLite), process maps, shared `reqwest::Client`, cached GitHub token
- `commands/sessions.rs`: PTY-based session spawning, throttled output (16ms batching via mpsc channel), prompt detection (`detect_prompt_pattern`), process group signals, graceful shutdown (SIGINT→SIGTERM→SIGKILL), 10MB output cap, session archiving
- `commands/shell.rs`: Separate shell terminal management with process group signals
- `commands/git_ops.rs` + `worktree.rs`: Git staging, commits, diffs, and worktree lifecycle
- `commands/github.rs`: GitHub API with shared HTTP client, token caching (5min), retry with exponential backoff, rate limit handling, CI checks, PR merge, issue close
- `commands/repos.rs`: Repository management operations
- `commands/ai.rs`: Claude API integration for session recaps, key-value settings store
- `commands/hooks.rs`: HTTP hook server receiving permission request events from Claude CLI
- `commands/filesystem.rs`: Slash command discovery with YAML frontmatter extraction
- `db.rs`: SQLite with WAL mode, `CREATE TABLE IF NOT EXISTS` schema init, `PRAGMA busy_timeout = 5000`, periodic WAL checkpoint
- `error.rs`: `AppError` with structured error codes (`DB_ERROR`, `IO_ERROR`, `HTTP_ERROR`, `NOT_FOUND`, `AUTH_FAILED`, `RATE_LIMITED`)
- `lib.rs`: Crash recovery (sentinel file, session recovery, orphaned worktree scan), prerequisites check, WAL checkpoint background task

### Session Status Flow
`running` → `attention` (needs user input or has issues) / `done` (completed, failed, or archived)
The `attention` status consolidates waiting, stuck, and orphaned states. `BlockType` discriminates waiting scenarios: `permission`, `confirmation`, `question`, `input`.

### Structured Claude UI
The frontend renders structured `ClaudeStreamEvent` JSON from the CLI rather than raw terminal output. `sessionStore` processes events into `ClaudeMessage` blocks (text, thinking, tool_use, tool_result) displayed by `ClaudeOutputPanel` and its child components in `src/components/claude/`. Permission requests from Claude CLI hooks are handled by `hookStore` → `PermissionDialog`/`PermissionBanner`.

### Database (SQLite)
Stored at `~/.octopus/octopus.db`. Tables: `repos` (GitHub URL, local path), `sessions` (linked to repo, tracks status, worktree path, linked issue/PR numbers, last_message, dangerously_skip_permissions), `settings` (key-value store for API keys etc.). Schema is created on startup via `db::create_schema()` using `CREATE TABLE IF NOT EXISTS`.

## Key Patterns

- **`useTauriEvent(subscribeFn, deps)`**: Handles async subscription with proper cleanup, including edge case where component unmounts before subscription resolves
- **`useAsync(factory, deps)`**: Returns `{ data, loading, error, reload }` with stale request cancellation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SebastianElvis/octopus](https://github.com/SebastianElvis/octopus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
