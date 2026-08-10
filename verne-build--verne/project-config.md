---
trigger: always_on
description: - **Backend**: Two Rust processes sharing one workspace crate (`portable-pty`, `rusqlite`, `tokio`, `tokio-tungstenite`, `git2`, `rmcp`, `notify`, `ignore`, `nucleo-matcher`). Talk to Electron main over unix sockets using a length-prefixed JSON protocol.
---

# Verne

## Stack

- **Backend**: Two Rust processes sharing one workspace crate (`portable-pty`, `rusqlite`, `tokio`, `tokio-tungstenite`, `git2`, `rmcp`, `notify`, `ignore`, `nucleo-matcher`). Talk to Electron main over unix sockets using a length-prefixed JSON protocol.
- **Frontend**: Electron, Vue 3 (composition API), Pinia, TailwindCSS v4, shadcn-vue (reka-ui), a custom canvas/WebGL2 grid terminal renderer (no xterm.js), Monaco
- **Speech**: sherpa-onnx (on-device STT) in a Node worker thread
- **Build**: electron-vite + electron-builder, TypeScript (strict), cargo

## Commands

```bash
pnpm dev          # electron-vite dev (debug daemon + bundled install run first via predev)
pnpm build        # electron-vite build
pnpm package      # release daemon build + electron-builder
pnpm typecheck    # vue-tsc --noEmit + tsc --noEmit
pnpm test         # vitest run
pnpm daemon:build:dev   # cargo build (debug)
```

Use **pnpm**, never npm.

### What Needs Restarting After a Change

`pnpm dev` runs `predev` first (`daemon:build:dev` → `cargo build` debug; then `bundle:install` → `pnpm install` for `resources/bundled` node deps — NOT the Rust binaries). In dev both Rust processes are spawned **directly from `daemon/target/debug/`** (`daemon-supervisor.ts`), so the cargo build IS the install. Note `predev` only runs when you *start* `dev` — saving a `.rs` file mid-session rebuilds nothing. What you must do depends on which layer you touched:

- **Renderer (Vue: `src/**`, components, composables, stores)** — electron-vite HMR hot-reloads. **No restart.**
- **Electron main / preload (`electron/main/**`, `electron/preload/**`)** — electron-vite restarts the main process automatically on save. If it doesn't catch, restart `pnpm dev`.
- **Rust sidecar code (git, db, file search/watch, settings, notes, hooks)** — does NOT live-reload. Stop `pnpm dev` and run it again: `predev` recompiles `daemon/target/debug/verne-sidecar` and the new app launch spawns it. The sidecar is `SIGTERM`'d on quit, so a clean stop is enough — but the supervisor *reuses an already-running sidecar* if its socket is still up, so make sure the prior instance actually exited. The persistent daemon survives, but it doesn't serve these methods, so no extra step.
- **Rust daemon code (PTYs, terminal grid ws bridge, tab spawn/kill, detection)** — the daemon is spawned **detached and survives app close**, so restarting `pnpm dev` alone keeps the *old* daemon running its stale binary. Rebuild (`pnpm daemon:build:dev`), then kill the detached daemon so a fresh one spawns: read the pid from the daemon pid file under `~/Library/Application Support/build.verne` (debug; see `paths.rs`) and `kill` it — don't broad-`pkill verne`, it'll match the dev server too. Then relaunch.

Quick rule: **frontend → nothing; sidecar → restart dev; daemon → kill the detached daemon then restart dev.** Unsure which process serves a method? Check `DAEMON_METHODS` in `electron/main/ipc-router.ts` (in the set → daemon; otherwise → sidecar).

## Architecture

Three layers: Electron main + renderer, plus two long-lived Rust processes.

```
renderer (Vue)  ──window.verne.invoke()──▶  Electron main  ──unix socket──▶  daemon  (verne)
       ▲                                          │         ──unix socket──▶  sidecar (verne-sidecar)
       └──────────── daemon-event ────────────────┘
```

1. **Electron main** (`electron/main/index.ts`) — owns the window, spawns/connects both Rust processes, registers native command handlers, installs the IPC router. Also hosts the bundled notes + browser **MCP server** (Node, `electron/main/mcp/`) that agents spawn via the `verne-mcp` launcher script. Privileged scheme registration (`verne-asset://`) MUST run before `app.whenReady()`.
2. **daemon** (`verne`) — spawned **detached**, **survives app close** so PTY sessions persist across reopen. Owns: PTYs, the terminal grid WebSocket bridge, agent detection snapshots, tab spawn/kill. Single-instance lock via pid file.
3. **sidecar** (`verne-sidecar`) — tied to the app lifecycle (killed on `before-quit`, uninstalls hooks). Owns: SQLite DB, git ops, file search/watch, settings, agent hooks, shadow repos, notes storage.

### IPC

**Renderer → main**: preload (`electron/preload/index.ts`) exposes `window.verne` via `contextBridge` (contextIsolation on):
- `invoke(method, params)` → `ipcRenderer.invoke("invoke", …)`
- `listen(event, cb)` → subscribes to `daemon-event` broadcasts
- `assetUrl(path)` → `verne-asset://local/<path>`

`src/platform/index.ts` wraps these as `invoke` / `listen` / `convertFileSrc` for the rest of the renderer. Always import from `platform`, not `window.verne` directly.

**Main → backends**: `electron/main/ipc-router.ts` routes each method:
- **Native** handlers (registered via `registerNative`): window, dialog, shell, menu, browser/CDP, LSP, metrics, review, speech, plus tab-orchestration shims.
- **daemon** methods: PTY ops, tab spawn/kill, detection snapshot, ws port, live tab ids.
- everything else → **sidecar**: git, db, file search, settings, notes, etc.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [verne-build/verne](https://github.com/verne-build/verne) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
