---
trigger: always_on
description: Tauri 2 desktop shell for `omp` (oh-my-pi). React UI served from `src/` in `tauri dev` (**no bundler** — JSX is transpiled in-browser by `@babel/standalone`) and from the precompiled `dist/` in release builds (see *Release frontend*). Rust backend spawns `omp --mode rpc` per tab.
---

# CLAUDE.md

Tauri 2 desktop shell for `omp` (oh-my-pi). React UI served from `src/` in `tauri dev` (**no bundler** — JSX is transpiled in-browser by `@babel/standalone`) and from the precompiled `dist/` in release builds (see *Release frontend*). Rust backend spawns `omp --mode rpc` per tab.

## Commands

| Task | Command |
|---|---|
| Install Tauri CLI | `npm install` |
| Dev | `npm run dev` |
| Prod build | `npm run build` (= `tauri build --config src-tauri/tauri.dist.conf.json`: embeds the precompiled `dist/`) |
| Build `dist/` only | `npm run build:frontend` (writes the gitignored `dist/`; also the last step of `npm test`) |
| Rust check (CI) | `cd src-tauri && cargo check --locked` |
| Rust fmt | `cd src-tauri && cargo fmt` |
| Rust lint (must stay clean) | `cd src-tauri && cargo +nightly clippy --all-targets --all-features -- -W clippy::pedantic -W clippy::nursery -D warnings` |
| Rust tests | `cd src-tauri && cargo test` |
| All JS regression scripts | `npm test` — add a new `test-*.mjs` to its chain in `package.json` (not `test-rpc.mjs`, which needs a live omp) |
| Probe omp RPC | `node test-rpc.mjs` |
| Keymap chord regression | `node test-keymap.mjs` (or `npm run test:keymap`) |
| Markdown XSS-escaping regression | `node test-markdown.mjs` (or `npm run test:markdown`) |
| Chat scroll-pin regression | `node test-scroll-pin.mjs` (or `npm run test:scroll-pin`) |
| Slash-command palette regression | `node test-slash-commands.mjs` (or `npm run test:slash-commands`) |
| Prompt history regression | `node test-prompt-history.mjs` (or `npm run test:prompt-history`) |
| Subagent manager reducer regression | `node test-subagents.mjs` (or `npm run test:subagents`) |
| Updater state regression | `node test-updater.mjs` (or `npm run test:updater`) |
| Updater feed assembly regression | `node test-updater-json.mjs` (or `npm run test:updater-json`) |
| Image viewer geometry regression | `node test-lightbox.mjs` (or `npm run test:lightbox`) |

`omp` must be on PATH (`%LOCALAPPDATA%\omp\omp.exe` on Win). CI and every release run the same suite (`.github/workflows/tests.yml`): `cargo test --locked` on win/linux/mac, plus `npm test`.

## Architecture

Three layers:

1. **Rust (`src-tauri/src/`)** — `agent/` module:
   - `mod.rs` — `AgentBridge` public API (start/stop/send/last_error).
   - `inner.rs` — `BridgeInner` per-session: generation token, `Arc<Mutex<ChildStdin>>`, child handle.
   - `spawn.rs` — `spawn_omp` candidate resolution + Win `CREATE_NO_WINDOW`.
   - `reader.rs` — stdout/stderr threads + bounded `read_until_capped` (16 MiB).

   `AgentBridge` = `HashMap<session_id, BridgeInner>`. Per-session stdin lock so writes don't serialise through the map. Reader emits `agent://line/{id}` per stdout line, `agent://exit/{id}` (empty payload = clean, non-empty = reason) — except a startup death before any frame ever arrived, where the reader substitutes a bounded stderr tail (`reader::StderrTail`) for the empty payload so a silent crash isn't read as a clean exit; also cached in `last_errors` so a background tab's death is visible from `session_status` without waiting for a switch. Tauri commands in `lib.rs`: `start_session`, `stop_session`, `send_command`, `session_status`, `open_project`, `take_pending_open_projects`. `Drop` + `stop_session` kill children — no orphans on hot-reload.

2. **Bridge (`src/live.js`)** — listens to `agent://line/{id}` for active session only. Holds per-session live state and a `sessionRegistry` (tabs). Tab switch: snapshot → tear down listeners → restore (or reset+`_initFetch`) → re-listen. Exposes `window.OMP_BRIDGE` (commands + `onUpdate`) and legacy `window.OMP_DATA`.

3. **React (`src/app-live.jsx` + `src/app/` + `src/design/*/`)** — sole React root. Uses `useBridgeSnapshot` (in `src/app/use-bridge-snapshot.jsx`) to mirror `OMP_BRIDGE.onUpdate` into hooks. Cross-cutting effects (theme on `<html>`) live there; keyboard shortcuts are resolved and dispatched by `src/app/use-keymap.jsx` (registry in `src/app/keymap.js`). Constants/framing strings in `src/app/constants.js`. Pure RPC↔UI shape transforms in `src/adapter.js` (no side effects, depends on `model-names.js`).

## Session model

One tab = one omp process. `default` session started in `lib.rs::setup`; new tabs via `OMP_BRIDGE.openSession(cwd)` → `start_session`. Tab switch preserves in-flight bubbles via `sessionSnapshots`; after re-listen, `get_messages` is called and `_handleResponse` merges persisted turns with cached `streamingBubble` (omp doesn't persist incomplete turns).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apoc/omp-desktop](https://github.com/apoc/omp-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
