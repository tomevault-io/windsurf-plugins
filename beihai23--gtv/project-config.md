---
trigger: always_on
description: **gtv** is a read-only desktop app that renders a local git repository's history as a
---

# AGENTS.md — gtv (Git Timeline Viewer)

## Project overview

**gtv** is a read-only desktop app that renders a local git repository's history as a
horizontal timeline of **branch lanes**: every branch gets its own track, is born at a
fork point, and folds back into its parent at a merge. It reconstructs branch lineage
from pure git data — no server, no metadata store, no account. Inspired by gmaster's
Branch Explorer, but an independent codebase.

gtv **never modifies the repository** it opens. Keep it that way.

## Tech stack

- **Desktop shell**: Tauri 2 (`src-tauri/`), app identifier `com.gtv.app`.
- **Backend**: Rust (edition 2021). Key crates: `git2` (0.20) for all git access,
  `tokio` (only used for `task::spawn_blocking` around git work), `serde`/`serde_json`,
  `chrono`, `thiserror`, `anyhow`, `log` + `env_logger`. The libgit2 bundled via
  `git2`/`libgit2-sys` must stay >= 1.9.4: older versions refuse to open repos whose
  config carries `extensions.relativeWorktrees` (written by git >= 2.48
  `worktree add --relative-paths`). Guarded by `tests/relative_worktrees_ext.rs`.
- **Frontend**: React 19 + TypeScript (strict) + Vite 7, D3 v7 for the timeline
  rendering (used directly, not via React wrappers — D3 owns the SVG DOM inside
  `Timeline.tsx`). Tauri plugins: `dialog`, `opener`.

## Repository layout

```
src-tauri/src/
  lib.rs          Tauri builder: plugins, AppState, invoke_handler (command registry)
  main.rs         thin entry point calling gtv_lib::run()
  models.rs       all shared data types (CommitNode, BranchLane, GitData, ...) —
                  the single source of truth for the IPC contract
  layout.rs       pure lane-propagation engine; NO git2 dependency, operates only
                  on models so it is unit-testable with hand-built graphs
  git_reader.rs   all git2 access: refs, chunked revwalk from branch tips
                  (walk_commits + load_more pagination), lazy diff stats;
                  feeds layout::compute_layout and returns GitData
  commands.rs     #[tauri::command] handlers + AppState (Mutex-guarded current
                  repo/path/branch/view + pagination ViewSession)
src-tauri/tests/
  layout_pure.rs  9 pure-graph algorithm tests (no git repo involved)
  tour_repo.rs    ground-truth benchmark against docs/reference/gmaster-tour
  relative_worktrees_ext.rs  regression: repos created by git >= 2.48
                  `worktree add --relative-paths` must open (libgit2 >= 1.9.4)
  pagination.rs   builds a temp repo via the git CLI and pages through it in
                  small chunks: paged result must equal the full walk, the
                  loaded set must stay downward-closed, and excluded stale
                  seeds must never be loaded
src-tauri/examples/
  dump_json.rs    dev tool: dump a repo's GitData as JSON
  dump_links.rs   dev tool: dump a repo's patch links (cherry-pick/rebase) as JSON
  dump_paged.rs   dev tool: page through a repo's ENTIRE history via load_more
                  and dump the final GitData (pagination stress test)
src/
  api.ts          thin wrappers around tauri invoke(), one per backend command
  types.ts        TypeScript mirror of models.rs — keep in sync by hand
  settings.tsx    Settings context: zh/en i18n dictionaries + preset theme
                  palettes (CSS custom properties applied to :root; App.css
                  consumes them via var(--x)), persisted in localStorage
                  (gtv_lang / gtv_theme / gtv_show_stale)
  App.tsx         top-level state: repo opening, branch panel, view options
  components/Timeline.tsx       the D3 timeline (lanes, edges, badges, minimap,
                                ruler, gestures) — ~1000 lines, the rendering core
  components/CommitDetails.tsx  commit detail panel
  components/SettingsDialog.tsx settings modal (Cmd/Ctrl+,): language, theme,
                                stale-branches toggle, About
mock.html         browser-only preview harness: mocks window.__TAURI_INTERNALS__
                  and feeds public/mock-data.json, so the frontend can be debugged
                  in a plain browser without the Rust backend
docs/
  roadmap.md / design-v2.md / gmaster-research.md   design docs (written in Chinese)
  reference/gmaster-tour    archived real git repo used as the test fixture
  reference/gmaster-io      archived gmaster website material (research only)
  tools/render_gitdata.py   script for rendering dumped GitData
```

## Build, run, and test commands

```bash
npm install                 # frontend deps (also pulls @tauri-apps/cli)
npm run tauri dev           # run the full desktop app (vite dev server + Rust)
npm run build               # typecheck (tsc) + vite production build → dist/
npm run tauri build         # produce a bundled desktop app

cd src-tauri
cargo test                  # ALL tests: pure layout tests + tour-repo benchmark
cargo test --test layout_pure   # just the fast pure-graph tests
cargo run --example dump_json -- /path/to/repo > public/mock-data.json
                            # regenerate the mock.html fixture from a real repo
```

There is no CI, no linter config, and no formatter config beyond the defaults.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [beihai23/gtv](https://github.com/beihai23/gtv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
