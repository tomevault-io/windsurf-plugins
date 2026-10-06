---
trigger: always_on
description: Minimalist live-preview markdown editor for macOS. Tauri 2 + Svelte 5 + CodeMirror 6.
---

# couplet (formerly md-mini)

Minimalist live-preview markdown editor for macOS. Tauri 2 + Svelte 5 + CodeMirror 6.

# md

## Commands

| Command | Description |
|---------|-------------|
| `npm install` | Install frontend dependencies |
| `npm run dev` | **Default for visual checks.** Frontend only (Vite at http://localhost:1420). No Tauri shell — open the URL in any browser. |
| `npm run dev:app` | Tauri dev with **renamed identifier** (`couplet-dev` / `pro.couplet.dev`). Own bundle ID, own single-instance socket, and own data directory (`~/Library/Application Support/couplet-dev/`) — does NOT conflict with an installed production app. Use when you need Tauri IPC (file I/O, menu, native dialogs). |
| `npm run check` | Svelte type checking |
| `npm run test` | Run frontend tests (vitest) |
| `cd src-tauri && cargo test` | Run Rust tests |
| `cargo clippy --manifest-path src-tauri/Cargo.toml` | Rust linter |
| `npm run build:dev` | **Renamed dev .dmg** (`couplet-dev`). Safe to install alongside production. |
| `npm run check:x86` | Compile for `x86_64-apple-darwin`. Fast guard that an Intel build still works — no bundle, no install. |
| `npm run build:universal` | **Production universal .dmg** (Apple Silicon + Intel). Same owner-triggered rule as `tauri build`. |
| `npm run tauri build` | **Production build (creates .dmg).** Replaces the user's installed couplet if installed. **Explicit, owner-triggered only — do NOT run as part of routine dev/QA.** |
| `npm run tauri dev` | Tauri dev with **production identifier**. Conflicts with running production couplet (shares single-instance socket, app data, recovery files), and refuses to start at all while an installed md-mini's data is still unmigrated (`migration.rs`, B3). **Use `npm run dev` or `npm run dev:app` instead.** |
| `npm run build` | Frontend build only (no Tauri bundle) |
| `npm run dev:site` | Dev server for the **couplet.pro landing** (Vite at http://localhost:1421). Never run alongside `npm run dev` — see the two-servers gotcha below. |
| `npm run build:site` | Build the landing from `site/` into `docs/`. Run `rm -rf docs/assets` first; read `site/CLAUDE.md` before touching the landing. |

### Build / Test Policy While User Is Working

- **Default for visual verification:** `npm run dev` (browser). Most couplet features are frontend; the editor, CodeMirror, decorations, mermaid all run in the browser context.
- **When Tauri features are needed:** `npm run dev:app`. Different identifier → different bundle ID → independent of the installed production app.
- **NEVER run `npm run tauri dev` or `npm run tauri build` as part of routine development or QA.** Both share state with the installed production app and can disrupt the user's active session. Production build is an **explicit owner-triggered action**, never an automated step.

## Architecture

```
src-tauri/src/          # Rust (Tauri backend)
  lib.rs                # App builder, plugin registration, setup
  main.rs               # Entry point
  commands.rs           # IPC commands: read_file, write_file, file_exists
  atomic_write.rs       # The ONE hardened file replace (mode/owner/ACL/xattr preserved, fsync, hidden temp). Used by commands.rs AND comments.rs — never fork it
  menu.rs               # macOS native app menu (Window → Minimize owns ⌘M)
  window.rs             # Window creation with cascade + file dedup; get_window_init (what a window shows on mount); PendingFiles; FileWatchers (records the watched path); reveal_other_window; window_set_number (the notch's rename); window_reveal_number (⌃1…⌃9)
  menu_route.rs         # Which window a menu action reaches: preferences to all, document actions to the last-focused one
  tabs.rs               # Tab registry behind OpenFiles: window → #N, ordered tabs, active; one file = one tab app-wide; check_move/move_tabs (all or nothing, tombstones the source)
  tab_commands.rs       # Tab IPC (tab_owner/open/claim/release/activate/close), decided under the registry lock; tab_move (plan 05: registry + AiPending + hand-over in one step) and tab_carousel_windows
  routing.rs            # Agent routing (spec §5): binding → open tab → project window (last focused) → empty main → new window; window projects; ls listing; carousel_rows (MRU, head of the active document)
  typing.rs             # App-wide "the human is typing" (Q10): note_typing IPC, last typed window + instant, blocks_raise for agents
  path_norm.rs          # The one spelling of a path (./.. and symlinks resolved), applied at every entry that takes a path from outside
  window_numbers.rs     # #N: lowest free in 1–99 among live windows; nothing persisted
  git_info.rs           # Project + branch for a path, read from .git/HEAD (no git process); tab_git_info for the drawer's grey line
  closed.rs             # Recently-closed tab stack (Cmd+Shift+T); saved files only, back into their window while it lives. ⌘⇧T never restores the session: File → "Reopen Windows from Last Session" (no key) does (Q1)
  recent.rs             # Recent Files list (recent.json), versioned broadcasts to every window; managed after paths::init
  paths.rs              # App data directory, named after the product (isolates dev from release)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [malinborn/couplet](https://github.com/malinborn/couplet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
