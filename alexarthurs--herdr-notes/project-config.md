---
trigger: always_on
description: A single herdr plugin: a persistent markdown notes pane (one scrollable note
---

# herdr-notes

A single herdr plugin: a persistent markdown notes pane (one scrollable note
per workspace, preview/edit modes). Standalone Rust crate — the repo root IS
the plugin root (`herdr plugin link .` from here).

**Living doc**: when you discover a non-obvious herdr/Windows/TUI behavior the hard
way, record it in the Gotchas section below before finishing the task. The fuller
findings doc (and the reference implementation, `herdr-sidebar`) lives in
`C:/Users/Alex/Projects/herdr/CLAUDE.md` — read it before deep herdr integration work.

## Layout

- `src/main.rs` — event loop (500ms poll); `--launch-decision` / `--focused-pane` /
  `--open-plan` stdin modes used by the launcher scripts
- `src/app.rs` — App state: preview/edit, clear-confirm overlay, 2s debounced
  autosave, 5s heartbeat, scrollbars
- `src/markdown.rs` — hand-rolled renderer (headings, lists, checkboxes, quotes,
  code, bold/italic, hr) + display-width wrapping
- `src/state.rs` — `{text, mode}` JSON, one file PER WORKSPACE in herdr's
  plugin state dir (`HERDR_PLUGIN_STATE_DIR/<workspace-id>.json`, e.g.
  `%LOCALAPPDATA%\herdr\plugins\herdr-notes\` — the docs-mandated home for
  durable plugin state; actions get the env var and the launchers pass it
  into the pane via `pane split --env`, the unix `[[panes]]` entry gets it
  natively). Without it (binary run by hand): config-dir fallback
  `%APPDATA%\herdr\notes\<workspace-id>.json` (unix:
  `$XDG_CONFIG_HOME|~/.config/herdr/notes/`); the state dir migrates
  fallback-layout files in on first load. Keyed by the stable
  `HERDR_WORKSPACE_ID` herdr injects into every pane (survives workspace
  renames; a closed workspace leaves a harmless orphan file). Unset or
  filename-unsafe (non-alphanumeric) id → legacy single-note
  `herdr/notes.json`; first per-workspace load MOVES a lingering legacy
  file into the workspace's slot (read-in-place if the rename fails; the
  per-workspace file wins when both exist). `note_key` exposes the note-FILE
  identity of a workspace id (None = shared legacy file; Windows folds ASCII
  case because NTFS filenames are case-insensitive) — the launcher guard
  compares THESE keys so it can never drift from the on-disk layout.
  Forgiving parse, atomic save (temp + `sync_all` + rename); path logic
  takes an injected base dir so tests never touch the real APPDATA
- `src/launch.rs` — OPEN/FOCUS/CLOSE/REPLACE toggle decisions (20s stale
  heartbeat OR label-without-token restart corpse → REPLACE); prefers a
  same-tab Notes pane but matches any pane whose
  `note_key` EQUALS the focused pane's, so a second instance on the same note
  file is never spawned (two live instances = last-writer-wins data loss) even
  when different raw workspace ids coarsen to one file (unsafe/missing ids →
  legacy, NTFS case folding); Notes panes on other note files are ignored
- `src/ipc.rs` — socket client: named pipe `\\.\pipe\<HERDR_SOCKET_PATH>` on
  Windows, unix socket elsewhere; one NDJSON request per connection
- `scripts/open-notes.ps1` / `open-notes.sh` — toggle launchers (right-dock);
  Windows entry goes through the inline-powershell action in `herdr-plugin.toml`

## Build / test / lint

```
cargo build --release
cargo test
cargo clippy --all-targets -- -D warnings
```

All three must pass before shipping. `cargo build --release` fails with os error 5
while the TUI is running in a pane — quit/close the pane first (and
`Get-Process herdr-notes | Stop-Process` for stragglers).

## Plugin dev workflow

- `herdr plugin link .` registers this checkout; `herdr plugin list --json` shows it.
- Open/toggle: `herdr plugin action invoke herdr-notes.open-notes-windows`
  (unix: `herdr-notes.open-notes`).
- `herdr plugin log list --plugin herdr-notes` shows action/spawn logs.
- After a rebuild, close any open Notes pane and re-invoke the action (stale panes
  keep the old binary).
- End-to-end verification: drive the real binary in a throwaway pane —
  `herdr pane split` + `pane run` + `pane send-keys` + `pane read --source visible`,
  then check `%APPDATA%\herdr\notes\<workspace-id>.json` (the pane's
  `HERDR_WORKSPACE_ID`). Cheap, catches what unit tests can't.

## Gotchas (verified against herdr 0.7.1)

Inherited from the sidebar plugin's findings:

- Windows herdr can NOT spawn a relative `[[panes]]` command (resolves against
  herdr's own dir) — Windows launches go through the action's inline powershell,
  which locates the plugin root via `herdr plugin list --json` (strip the `\\?\`
  prefix) and spawns the exe by absolute path.
- Action ids must be globally unique across platforms — hence the `-windows`
  suffix, both variants gated by the item-level `platforms` key.
- herdr panes run Windows PowerShell 5.1: chain with `;` / `if ($?)`, never `&&`.
  PS 5.1 prepends a UTF-8 BOM when piping into a native exe's stdin — everything
  parsing herdr JSON from stdin strips a leading `\u{feff}` (see `state.rs`/`launch.rs`).
- `pane split --ratio` is the ORIGINAL pane's share (the new pane gets 1 − ratio);
  ratios clamp to a 0.1 floor.
- Metadata token values must be STRINGS (numbers rejected silently); the heartbeat
  token (`herdr-notes` = unix-time string) re-stamps every ~5s so launchers can
  tell a live pane from a corpse (>20s stale → REPLACE).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexarthurs/herdr-notes](https://github.com/alexarthurs/herdr-notes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
