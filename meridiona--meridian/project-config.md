---
trigger: always_on
description: Meridian is a single-process Rust daemon that normalises raw screen-capture frames into structured, app-based activity sessions stored in its own SQLite database at `~/.meridian/meridian.db`. A Next.js dashboard and a TypeScript MCP server sit alongside the daemon. (Capture source: historically screenpipe's SQLite DB; since the Bucket-2 cutover on `feat/in-process-capture` the frames are produced **in-process by the tray** and the daemon reads `meridian.db`'s own capture tables — see "Capture so
---

# Meridian — Claude Code Instructions

Meridian is a single-process Rust daemon that normalises raw screen-capture frames into structured, app-based activity sessions stored in its own SQLite database at `~/.meridian/meridian.db`. A Next.js dashboard and a TypeScript MCP server sit alongside the daemon. (Capture source: historically screenpipe's SQLite DB; since the Bucket-2 cutover on `feat/in-process-capture` the frames are produced **in-process by the tray** and the daemon reads `meridian.db`'s own capture tables — see "Capture source — in-process" below.)

---

## Hard Rules

- Do what has been asked; nothing more, nothing less
- NEVER create files unless absolutely necessary — prefer editing existing files
- NEVER create documentation files unless explicitly requested
- ALWAYS read a file before editing it
- NEVER commit secrets, credentials, or `.env` files
- Keep files under 500 lines; split when a file grows beyond that
- Validate all input at system boundaries (config load, DB open, frame parsing)
- NEVER run `git reset`, `git push --force`, or delete local code — other agents may be working on the codebase in parallel
- NEVER merge a PR automatically — open/update PRs as needed, but leave the actual merge to a human reviewer
- NEVER push directly to `main` — always create a separate feature branch, commit there, and raise a PR to `main`
- ALWAYS use a separate branch per feature/fix — branch name format: `type/short-description` (e.g. `feat/trello-oauth`, `fix/ui-disconnect`)

---

## File Header Requirement

Every `.rs`, `.ts`, and `.tsx` file must start with this comment as its very first line:

```
//ambient dev tool that watches what you do and updates your PM tickets automatically, boosting developer productivity
```

SQL migration files use the SQL comment form:

```
-- ambient dev tool that watches what you do and updates your PM tickets automatically, boosting developer productivity
```

The `commit-msg` hook enforces conventional commit format. The `pre-commit` hook enforces `cargo fmt` and `cargo clippy`. The `pre-push` hook runs the full suite: fmt + clippy + `cargo test` + UI build + UI tests.

---

## Repository Layout

```
meridian/
  src/
    main.rs              # daemon entry point — tokio::main, signal handling, poll loop
    lib.rs               # public crate root
    config.rs            # Config::from_env() — reads env vars, expands ~
    db/
      mod.rs
      meridian.rs        # writes app_sessions, active_session, etl_runs, etl_cursor, gaps
      screenpipe.rs      # read-only queries against screenpipe's frames/ocr/audio/ui_events
    etl/
      mod.rs
      runner.rs          # run_etl() — batch loop, gap detection, block state machine
      extractor.rs       # extract_block_context() — OCR, audio, signals, window titles
    migrations/
      001_initial.sql    # app_sessions, active_session, etl_runs, etl_cursor
      002_gaps.sql       # gaps table, idle_frame_count columns
  meridian-core/         # lean shared data layer — used by BOTH the daemon and the Tauri dashboard
    src/
      lib.rs             # thin manifest: declares modules + curated `pub use` re-exports (stable public API)
      db.rs              # ActiveSession + open_existing + get_active_session (daemon re-exports these)
      settings.rs        # settings.json runtime config reader (daemon re-exports)
      util/              # DB-free helpers, re-exported flat (meridian_core::{intervals,date,hygiene})
        intervals.rs     # wall-clock interval math (port of ui/lib/intervals.ts)
        date.rs          # local-day bounds (port of ui/lib/date-utils.ts)
        hygiene.rs       # board-hygiene reason → hint/fix mapping
      readers/           # the ported /api/* DB readers, re-exported flat (meridian_core::today, ::tasks, …)
        active.rs  coding_agents.rs  integrations.rs  tasks.rs  triage.rs  week.rs  worklogs.rs
        today/           # mod.rs + types.rs (size split — types co-located per module)
  tests/
    integration_etl.rs   # integration tests — in-memory SQLite, no network
  ui/
    app/
      layout.tsx         # root layout
      page.tsx           # dashboard home
      sessions/          # session list and detail pages
      apps/              # per-app breakdown pages
      api/               # Next.js route handlers (active, sessions, stats, timeline)
    components/          # ActiveSessionCard, AppTable, DayTimeline, FocusDonut, Nav, …
  packages/
    meridian-mcp/        # TypeScript MCP server — exposes meridian.db to AI clients
      dist/index.js      # compiled output (committed)
  tray/
    src-tauri/           # Tauri shell (Rust + Tauri framework)
      src/
        main.rs          # Tauri entry point
        lib.rs           # thin app bootstrap (builder, db pool, tray install, invoke_handler)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Meridiona/meridian](https://github.com/Meridiona/meridian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
