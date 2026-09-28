---
trigger: always_on
description: Tauri 2 (v2.11.1) + Rust backend, vanilla JS frontend. No bundler, no `package.json`, no Node build step. Version `2.2.6` in `Cargo.toml` and `tauri.conf.json`. Identifier `com.monoloth.app`. Cross-platform (Windows/macOS/Linux). Rust 1.77.2+ and C++ Build Tools required.
---

# AGENTS.md

## Stack

Tauri 2 (v2.11.1) + Rust backend, vanilla JS frontend. No bundler, no `package.json`, no Node build step. Version `2.2.6` in `Cargo.toml` and `tauri.conf.json`. Identifier `com.monoloth.app`. Cross-platform (Windows/macOS/Linux). Rust 1.77.2+ and C++ Build Tools required.

## Layout

- `src-tauri/src/` — `main.rs` → `lib.rs::run()` registers plugins, restores window state, wires close handler, lists all IPC commands. Commands in `commands/` (one file per concern: `config`, `fs`, `history`, `image`, `profile`, `shell`, `terminal`, `version`, `window`). Core: `pty.rs` (PTY session manager), `config.rs` (untyped `serde_json::Value` map via `Arc<Mutex<ConfigInner>>`), `history.rs`.
- `frontend/` — `index.html` loads `<script>` tags in **load-bearing order** below. IIFE modules expose one `window.Monolith*` global each — no imports. `lib/` has vendored `xterm*`, `dom-utils.js` (sets `window.MonolothUI`), and plugin wrappers.
- `style.css` (~6991 lines) — flat CSS, no preprocessor. `--modal-*` CSS variable tokens for themeable colors.
- Tests: 14 `*.test.cjs` suites / 90 tests in `frontend/` using Node `vm` sandbox. `cargo test` for Rust (80 tests across config, terminal, history, fs, image).

## Build / test commands

No scripts. Raw commands:
```bash
# from repo root
cargo check --manifest-path src-tauri/Cargo.toml   # fast typecheck
cargo test --manifest-path src-tauri/Cargo.toml    # all Rust tests (80)
node --test frontend/*.test.cjs                    # all frontend tests (90)
node --test frontend/terminal.test.cjs             # single suite

# from src-tauri/ (workdir, not --manifest-path — CLI is npm's tauri.cmd)
tauri dev                                          # dev build with hot-reload
tauri build                                        # release build
```
`cargo test --manifest-path src-tauri/Cargo.toml` works from repo root. `node --test` must run from repo root — suites hardcode repo-root-relative reads. `cargo tauri` is not installed; use `tauri` (npm's `tauri.cmd`) from `src-tauri/`. Neither `tauri dev` nor `tauri build` accepts `--manifest-path`.
No formatter, linter, or pre-commit. Don't introduce one without asking.

## Frontend load order (load-bearing)

`xterm*` → `tauri-bridge.js` (`window.monolithApi`) → `dom-utils.js` (`window.MonolothUI`) → `ctx-menu.js` (`window.MonolithCtxMenu`) → `plugin-updater.js` → `plugin-process.js` → `updater-toast.js` → `tooltip.js` → `shortcuts.js` → `theme.js` → `dialog.js` → `file-picker.js` → `command-palette.js` → `profiles.js` → `lib/terminal-view.js` → `terminal.js` → `app.js` → `sidebar.js`

Do not reorder. Cache busters (`?v=N`) on every `<script>` and `<link>` tag — WebView2 caches aggressively. Bump ALL when you change any file.

## Frontend module architecture

Each IIFE module (loaded before `app.js`) exposes ONE `window.Monolith*` (or `Monoloth*`) global. Modules communicate ONLY through `window` globals — no imports.

- `MonolothUI` (lib/dom-utils.js) — DOM helpers shared by all modules (note: `Monoloth` spelling)
- `MonolithCtxMenu` (ctx-menu.js) — shared context menu factory, icons, dismiss logic
- `MonolothTooltip` (tooltip.js) — tooltip positioning/lifecycle (note: `Monoloth` spelling)
- `MonolithShortcuts` (shortcuts.js) — parse/match/load/save shortcuts
- `MonolithTheme` (theme.js) — theme/CTA state, `applyTheme`/`applyCtaStyle`/`syncOutlineOnLightClass`, xterm palettes
- `MonolithDialog` (dialog.js) — `showPrompt`/`showConfirm`
- `MonolithFilePicker` (file-picker.js) — `pickPath(opts)`
- `MonolithPalette` (command-palette.js) — palette nav, open/close/filter
- `MonolithProfiles` (profiles.js) — profile CRUD + switcher modal
- `MonolithTerminal` (terminal.js) — xterm session lifecycle, sets `window.writeToTerm` and `window.__monolithTermWinOpts`
- `MonolothApp` (app.js) — facade exposed on `window.MonolothApp` for other modules and sidebar.js. Coordinates background config, settings, bootstrap/reveal, keydown handler, history, recent-dirs, titlebar. Sets `window.__monolithWindowsPty`.

**Rules:**
- A module may reference `window.MonolothApp.*` or another module's global ONLY inside functions/handlers (event time), NEVER at IIFE top-level — `app.js` loads LAST.
- External contracts: Rust backend calls `window.writeToTerm`; `sidebar.js` calls `window.__monolithTermWinOpts()` and `window.MonolothApp` methods.
- The shared `keydown` handler stays in `app.js`, delegates to `MonolithPalette`/`MonolithDialog`/`MonolithProfiles` via their `is*Active()` + action methods.

## Tauri quirks

- `withGlobalTauri: true` — use `window.__TAURI__.core.invoke` for IPC. All calls are async.
- `app.windows[0].visible: false` in config. Window is shown via `window.show()` in `lib.rs:133`. Do not flip to `true`.
- Updater endpoint: `https://github.com/noahain/Monoloth/releases/latest/download/latest.json`. The `Monoloth` spelling is canonical (yes, typo). Do not "fix" it.
- `plugins.updater.pubkey` is a real signing key. Private key (`TAURI_SIGNING_PRIVATE_KEY`) is a CI secret; keep it safe.

## Bridge response wrapping


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [noahain/Monoloth](https://github.com/noahain/Monoloth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
