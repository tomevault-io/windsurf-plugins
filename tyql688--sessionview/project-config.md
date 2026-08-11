---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) and other agents working in this
---

# SessionView

Guidance for Claude Code (claude.ai/code) and other agents working in this
repository. `CLAUDE.md` includes this file via `@AGENTS.md`.

SessionView is a desktop app that brings local AI coding sessions from many
tools — Claude Code, Codex, Antigravity, Kimi Code, Cursor, OpenCode, CC-Mirror,
Pi, and Grok Build — into one place to read, search, analyze usage, export, and
resume.
Stack: Tauri 2 + React 19 (with React Compiler) + Rust + SQLite (FTS5).
Enforcement-mapped coding standards live in `style/ts.md` and `style/rust.md`;
when this file and those disagree, those win.

## Commands

```bash
# App / frontend
npm run tauri dev        # run the app with hot reload
npm run tauri build      # production bundle

# Headless server (browser shell; build dist first so it can be embedded)
npm run build && cd src-tauri && \
  cargo build --release --no-default-features --features headless
# gates must pass under BOTH feature sets: default (gui) and
# --no-default-features --features headless
npm run check            # typecheck + Biome + ESLint — the frontend gate
npm test                 # frontend tests (Vitest, run mode)
npm run knip             # dead-code / dependency audit — a release gate

# Rust backend (run from src-tauri/)
cargo test               # backend tests
cargo fmt --check        # format check
cargo clippy --all-targets --all-features -- -D warnings   # lint, CI-strict

# Release
./scripts/release.sh <version>   # bump, commit, tag, push -> CI release
```

Run a single test:

```bash
npm test -- src/lib/foo.test.ts               # one frontend file
npx vitest run -t "resolves subagent"         # frontend, by test name
cd src-tauri && cargo test parent_backfills   # one Rust test by name substring
cd src-tauri && cargo test --test parser_tests# one Rust integration-test file
```

Real-data smoke tests (`src-tauri/tests/*_real_*.rs`) are `#[ignore]` by default;
run them explicitly with `cargo test -- --ignored`. Git hooks (lefthook)
pre-commit format staged frontend files and run ESLint; pre-push runs
`npm run check` + `npm test` and the Rust fmt/clippy/test gate. CI and
`scripts/release.sh` are the source of truth for exact commands.

## Architecture

The core job: normalize many tools' on-disk session logs into one queryable
model, then render, search, and analyze them. Understanding it means reading
across `src-tauri/src/{providers,provider,indexer.rs,db,models.rs}` and
`src/{features,stores}`.

### Backend pipeline (`src-tauri/src/`)

- **Providers** (`providers/`) each implement the `SessionProvider` trait
  (`provider/traits.rs`) and parse one tool's logs into a normalized
  `ParsedSession` / `Message` model (`models.rs`). A provider is authoritative
  for its own fields. Provider identity and metadata are bridged through a
  `Provider` enum + descriptor (`provider/`), so **adding a provider is a
  cross-layer change**, not just a parser: enum + catalog + Tauri asset-scope
  allowlist + frontend provider type + theme/snapshot fallback + resume
  behavior + tests. Exhaustive `match` on `Provider` makes the compiler surface
  most of these.
- **Indexing** (`indexer.rs`) is incremental: providers short-circuit unchanged
  files by `(size, mtime)` via `scan_incremental`, and a `maintenance_running`
  guard (`commands/sessions.rs`) serializes passes so a scan never races the
  app. Parsed sessions upsert into **SQLite** (`db/`) with FTS5 backing
  full-text search; progress streams to the UI as `maintenance-status` events.
- **Commands** (`commands/`) are the backend surface and the trust boundary —
  validate provider strings and path inputs here, and keep the asset scope
  allowlist-based. Command bodies are transport-agnostic core functions
  (`fn(state: AppState, …)`); two thin shells wrap them: `commands/gui.rs`
  (`#[tauri::command]`, feature `gui`, default) and `server/dispatch.rs`
  (HTTP invoke, feature `headless`). Events go through the `EventBus` trait
  (`services/events.rs`) — Tauri emit in the GUI, SSE broadcast headless.
  **Adding a command — or changing its signature — means updating all four:
  the core, the gui wrapper + `generate_handler!` list, the dispatch match
  arm, and the `BackendCommandMap` entry in `src/lib/tauri.ts`.** Every usage
  query also takes an optional IANA `timezone`; the frontend sends the
  viewer's zone, so a remote headless client gets its own day boundaries.
- **Headless shell** (`server/`, feature `headless`): axum server (default
  port 9921) serving the embedded `dist/` plus `POST /api/invoke/{command}`,
  `GET /api/events` (SSE), and export-download endpoints. It shares the GUI's
  data dir and SQLite index (WAL + busy_timeout make the two processes safe
  to run concurrently), so it never re-indexes what the GUI already did — a
  schema change is the exception: the first process on the new schema drops
  the derived stats and resets `source_mtime`, forcing one full re-index, and
  running mixed binary versions against one data dir is not supported. The
  frontend picks its transport at runtime via `src/lib/runtime.ts`
  (`__TAURI_INTERNALS__` detection); `npm/` holds the `npx sessionview`
  launcher and platform-package generator.
- **Parent/child trees.** Subagents and sidechains are child sessions, linked to

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tyql688/sessionview](https://github.com/tyql688/sessionview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
