---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Token Dashboard** — a local desktop dashboard for tracking Claude Code token usage, costs, and session history. Reads the JSONL transcripts Claude Code writes to `~/.claude/projects/` and turns them into per-prompt cost analytics, tool/file heatmaps, subagent attribution, cache analytics, project comparisons, and a rule-based tips engine.

Inspired by [phuryn/claude-usage](https://github.com/phuryn/claude-usage) but diverges in UI (React 18 bundled with esbuild, dark theme, hash router, SSE refresh) and scope (expensive-prompt drill-down, skills view, tips engine, streaming-snapshot dedup). See `docs/inspiration.md` for the original's feature set.

## Status

**5.0 line — Rust + Tauri.** The 3.x Python + Electron stack is no longer in the tree; existing 3.x users keep their installed builds, future development targets v5 only. Workspace builds clean on `cargo build --workspace`; ~180 tests across `core` + `cli` (run `cargo test --workspace` for the live count). Tauri shell verified on Windows; macOS and Linux QA happens via the release-tauri pipeline.

## Architecture

```
crates/
  token-dashboard-core/   scanner, db, queries, pricing, preferences,
                          tips, skills_catalog, anthropic_sync, sources.
                          No process model — just a library.
  token-dashboard-cli/    axum router + tokio bin (`token-dashboard`).
                          Owns the /api/* surface and the SSE bus.
  token-dashboard-tauri/  Tauri 2 desktop shell. Links the cli as a
                          library and calls app(state) directly inside
                          the tauri runtime — single process. The
                          analytics surface spawns no subprocess. The
                          src/live/ module (ported from Praetorium) is
                          the one exception: it drives the `claude` CLI
                          (process.rs) and watches ~/.claude/projects/
                          (session_watch.rs), streaming events to the
                          frontend over Tauri IPC channels.
  praetorium-core/        Pure parser/vault library (no Tauri, no db,
                          no async runtime) backing the Live feature.

frontend/                 React 18 + esbuild (`entry.jsx` →
                          `frontend/dist/app.js`). The webview talks
                          directly to /api/* through the embedded
                          server; the SSE consumer in api-client.js
                          drives view refreshes.
docs/                     plan, design, roadmap, inspiration.
```

## Data source

Claude Code writes one JSONL file per session to `~/.claude/projects/<project-slug>/<session-id>.jsonl`. Each line is a message record; usage fields live at `message.usage` and model identifier at `message.model`. The scanner (`crates/token-dashboard-core/src/scanner.rs`) is incremental — it tracks each file's mtime and byte offset in the `files` table and only reads new bytes on subsequent scans.

## Conventions

- **Fully local.** No telemetry, no remote calls for user data. Tests run offline. **Exception:** the user-initiated `POST /api/limits/sync` route makes one Anthropic API call with the user's saved key to read rate-limit headers; this is opt-in, never automatic, and disabled until the user saves a key in Settings.
- **rusqlite parameter binding always.** Any `format!()` in a SQL statement must interpolate only internal, caller-controlled values (column names, placeholder lists). User-reachable values go through `?` via `params!` or `params_from_iter`.
- **Streaming-snapshot dedup.** When adding scanner logic that joins the `messages` table, remember `(session_id, message_id)` is the dedup key, not `uuid`. See `scanner::evict_prior_snapshots` and the migration note in `db::migrate_add_message_id`.
- **Small files with clear responsibilities.** If a file grows past ~600 lines or accretes three distinct concerns, split it.

## Tauri build prereqs

The frontend bundle must exist at `frontend/dist/app.js` before `cargo run -p token-dashboard-tauri` or `cargo tauri build`. Run `cd frontend && npm install && npm run build` once. (Tauri's `beforeBuildCommand` hook is intentionally omitted: Tauri 2 picks `frontend_dir` by walking up from `tauri.conf.json` for a `package.json`; this repo has none at the workspace root, so the hook resolves outside the tree. CI's `release-tauri` workflow pre-builds the frontend explicitly.) The shell walks upward from the binary to find `frontend/index.html`; override with `TOKEN_DASHBOARD_STATIC` when bundling for distribution.

## Frontend cache (avoid re-reading hot files)

**`frontend/styles.css`** — single bundled stylesheet, do not split (esbuild order matters).
- Root scope: `.dir-a-root` (everything is namespaced under it). Glass-mode toggle: `.dir-a-root.is-glass`.
- Component classes: `.a-card`, `.a-kpi`, `.a-kpi-row`, `.a-strip`, `.a-strip-{left,mid,right}`, `.a-topbar`, `.a-table`, `.a-sticky-head`, `.a-glass-slider`, `.a-metric`, `.a-pre`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Arylmera/Token-Dashboard](https://github.com/Arylmera/Token-Dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
