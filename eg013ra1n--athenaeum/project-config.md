---
trigger: always_on
description: Guidance for Codex working in the Athenaeum repo.
---

# AGENTS.md

Guidance for Codex working in the Athenaeum repo.

## Project Overview

Athenaeum is a desktop + web app for astrophotographers to manage FITS/XISF image files and their metadata catalog (frame-set clustering by sky coordinates, calibration matching, ZIP archive, plate-solving, export templates). Tauri 2 desktop shell, Axum/SSE web server, shared `athenaeum-core` library, SQLite catalog, React/TS frontend.
If you are using any other code base like ASTAP or other don't name it in the code and comments and do not name functions with its name.

## Workspace Layout

Cargo workspace + git submodule:

- `crates/athenaeum-core/` — shared library; all non-IPC logic (DB, FITS parsing, calibration, scanner, archive, file_op, analysis, plate_solve, services, …).
- `crates/athenaeum-tauri/` — desktop shell. `commands/` modules thinly wrap `athenaeum-core`.
- `crates/athenaeum-web/` — Axum HTTP/SSE server for the Docker/web build. `routes/` modules mirror Tauri commands one-for-one.
- `rustafits/` — git submodule (FITS image rendering); path dep of `core` + `tauri`.
- `src/` — React/TS frontend. `src/api/` abstracts Tauri IPC vs HTTP/SSE behind a single `api` object selected by `VITE_TARGET`.

## Critical Rules

- **Two backends in sync.** Adding/modifying a Tauri command (`crates/athenaeum-tauri/src/commands/<domain>.rs`) requires the matching Axum route (`crates/athenaeum-web/src/routes/<domain>.rs`) in the same change. Put real logic in `athenaeum-core`; the Tauri/Axum layer is a thin wrapper.
- **No `@tauri-apps/*` imports outside `src/api/`.** Frontend always goes through the `api` object.
- **Serde boundary: snake_case ↔ camelCase.** Use `#[serde(rename_all = "camelCase")]` and verify TS interfaces in `src/types/models.ts` match.
- **Never swallow errors.** Always log to console/stderr before returning; silent failures have repeatedly cost hours.
- **Minimal scope.** Don't over-engineer or build adjacent dependency trees unprompted. Ask if scope is unclear.
- **Real data first when debugging.** Synthetic tests can mask real-world bugs — switch to a real FITS file early.
- **Clarify domain terms.** Don't substitute (`equipment ID` ≠ `calibration set ID`, `filter` ≠ `sort`).
- **Design tokens, not raw colors.** Use `bg-surface`, `text-content-muted`, `bg-accent`, `text-error`, … so dark/light themes both work.
- **Multi-file edits in complete passes.** Avoid many small partial edits to large files.
- **`anyhow::Result`** inside core; convert with `.map_err(|e| e.to_string())` at the command boundary.

## Commands

```bash
# Desktop
npm run tauri dev          # Hot-reload desktop app
npm run tauri build        # Full desktop build

# Web / Docker
npm run dev:web            # Vite frontend, VITE_TARGET=web
cargo run -p athenaeum-web # Axum server locally

# Tests
cargo test --workspace     # All Rust crates
cargo test -p athenaeum-core
```

DB lives in OS app-data dir for desktop; `/data` (or `$ATHENAEUM_DB_PATH`) in Docker. Schema in `crates/athenaeum-core/src/db/schema.rs`.

## Module Map

**`athenaeum-core` (`crates/athenaeum-core/src/`)** — see `lib.rs` for the canonical list. Top-level domains: `models`, `coordinates`, `db`, `fits_parser`, `clustering`, `settings`, `scanner`, `monitor`, `duplicates`, `calibration`, `archive`, `file_op`, `export`, `analysis`, `plate_solve`, `cache`, `catalog`, `auto_merge`, `relinking`, `sessions`, `services` (`ServiceContext` + `ProgressEmitter` trait), `events`, `logging`, `rustafits_processor`.

**Tauri commands (`crates/athenaeum-tauri/src/commands/`)** — ~157 functions across 16 modules (measured 2026-08-04, post dead-code-cleanup cycle; `utils` was deleted — its last helper moved into core with the spatial migration. `cache` is an empty placeholder module post-T6 — still declared in `mod.rs` so it counts as a module, contributes 0 commands). Each has a sibling in `crates/athenaeum-web/src/routes/` with the same name and surface:

`core` `scan_roots` `files` `settings` `frame_sets` `calibration` `duplicates` `cache` `spatial` `archive` `analysis` `plate_solve` `registration` `export` `missing_files` `calendar`

Frontend pages live in `src/pages/`; routing in `src/App.tsx` (React Router v7, `/` → `/files`).

## Adding a Tauri Command

1. Put the logic in `athenaeum-core` (so both backends call it).
2. Add `#[tauri::command] pub async fn …` in the right `commands/<domain>.rs` (re-exported by `commands/mod.rs`), with `#[tracing::instrument(skip_all, err)]` directly beneath the command attribute (boundary span + never-swallow — see Logging). Web mirrors get the same attribute (`err(Debug)` when the error type is `(StatusCode, String)`; plain `skip_all` for non-Result handlers). Commands fired per-frame/per-index in UI loops add `level = "debug"`.
3. Register it in `commands::…` in `invoke_handler` in `crates/athenaeum-tauri/src/lib.rs`.
4. Mirror it in `crates/athenaeum-web/src/routes/<same_domain>.rs` and register in `routes/mod.rs`. For progress, use `SseProgressEmitter::new(state.event_tx.clone())`.
5. Call from React via `api.invoke('command_name', { args })` — never `@tauri-apps/api` outside `src/api/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eg013ra1n/athenaeum](https://github.com/eg013ra1n/athenaeum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
