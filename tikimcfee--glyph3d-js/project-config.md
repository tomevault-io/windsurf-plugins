---
trigger: always_on
description: glyph3d-js renders thousands of text glyphs in 3D at 60fps via GPU instancing on a
---

# CLAUDE.md — Project Guide for AI Assistants

## What this is

glyph3d-js renders thousands of text glyphs in 3D at 60fps via GPU instancing on a
WebGPU renderer — built for navigating source code, directory trees, and live
terminals in 3D space. The repo is organized as **a package, an app, and a cli**:

- **`packages/glyph3d-core`** (`@glyph3d/core`) — the publishable rendering library:
  glyph atlas + shaping, the `GlyphField` WebGPU renderer, `CodeGrid` / `TerminalGrid`,
  layout managers, picking, the command/interaction services.
- **`packages/glyph3d-r3f`** — react-three-fiber bindings for the core.
- **`app/`** — the IDE: a react-three-fiber client on the core. File tree + 3D code
  grids + terminals, driven by a command bus. `app/client/` holds the shared chrome
  (CommandProvider, CanvasInteraction, HudPanel, CommandBar, SessionStore,
  WorkspaceModel); `app/commands/` holds the command spine (handlers).
- **`cli/`** — a Go single binary that bakes in the built app and serves it alongside
  a WebSocket relay + sandboxed filesystem RPC.

## Tech stack

- **JavaScript** (ES modules), **JSDoc** for types — no TypeScript.
- **Three.js via `three/webgpu`**; shaders are **TSL** (Three.js Shading Language)
  NodeMaterials, not GLSL.
- **Bun workspace** (`packages/*`, `app`); **Vite** builds the app. The Go binary
  builds with `make`.
- **React 19 + @react-three/fiber 9** for the app.
- Glyph shaping: **HarfBuzz** + **Slug** GPU bezier coverage, with a font fallback
  chain and color-emoji support.

The IDE shell is a react-three-fiber application on a framework-agnostic rendering
core: `@glyph3d/core` is vanilla `three/webgpu` with no UI-framework dependency; the
shell is **React 19 + react-three-fiber 9**; the two meet at the thin `@glyph3d/r3f`
binding. New shell code is React/r3f; new rendering code is `three/webgpu` + TSL.

## Structure

```
packages/
  glyph3d-core/src/        @glyph3d/core
    GlyphField.js          WebGPU / TSL renderer
    GlyphAtlas.js          font atlas (shaping → glyph metrics)
    collections/           CodeGrid, TerminalGrid, layout managers
    core/                  LayoutDescription (layout seam), constants, renderOrder, types
    services/              interaction (AttentionManager, keyEncoding),
                           camera (ViewerCameraController), data, orchestration
                           (CommandRouter, WebSocketBridge), state, visual
    picking/  shaping/  workers/  annotations/  parsing/  hand/  fonts/
  glyph3d-r3f/src/         GlyphCanvas, ViewerCamera, useGlyphEngine, useGridRegistry
app/
  main.jsx index.html vite.config.js   the IDE entry (Vite)
  client/                CommandProvider, CanvasInteraction, HudPanel, CommandBar,
                         SessionStore, WorkspaceModel
  commands/handlers/     command verbs (file.*, grid.*, edit.*, terminal.*, camera.*, …)
  ButtonBar IdeDock FileTree TerminalsPanel AgentsPanel CarrelsPanel   dockview panels
cli/
  main.go relay.go fs.go embed.go attach_unix.go   serve + relay + fs-RPC + terminals
Makefile  tools/dev.sh
```

## How it runs

**Dev loop** (the `webgpu-dev-loop` skill): `tools/dev.sh` runs Vite (`:5173`) + the Go
relay (`:8080`). Open `http://localhost:5173` and **hard-reload** (Ctrl-Shift-R) after
a Vite restart. Use a WebGPU browser — Chromium-based (Vivaldi/Chrome) or the
NVIDIA-pinned `tools/dev-firefox.sh`; plain Firefox on Linux crashes on WebGPU.

- `tools/dev.sh vite` restarts Vite + clears its cache (do this after editing
  core/handlers — the watcher can serve stale transforms; verify with
  `curl http://localhost:5173/@fs/<abspath> | rg <token>`).
- `tools/dev.sh relay` rebuilds + restarts the Go relay.

**Build / serve**: `make build` runs the app's Vite production build, stages
`app/dist/` into `cli/web/`, and bakes it into the binary. `./glyph3d-cli serve
~/project` then serves the built IDE + relay + fs-RPC from one binary at `/`. The app
bundles its deps at build (no importmap). `make all` cross-compiles all platforms.

**Drive it from the CLI** — the same command bus the UI uses:
`./glyph3d-cli grid.list`, `./glyph3d-cli file.open path/to/file.js`. Global flags go
*before* the subcommand (e.g. `--port 8099`).

## Command bus

Every action is a verb. `CommandRouter.execute(input)` dispatches to a handler in
`app/commands/handlers/`; UI clicks and CLI/RPC hit the same handlers (one source of
truth). The relay forwards CLI commands to the browser display over WebSocket, and
keeps a structured in-memory SQLite store of every browser log record — `log.query` /
`log.search` / `log.errors` / `log.stats` / `log.dump` answer relay-side (page-less),
with live follow via `bun tools/buslog.mjs`. Page-side, `emitLogRecord` runs a storm
brake: an identical error/warn signature past 5 is dropped + counted, with a
`log-brake` summary warn every 500 (locked by `tools/log-storm-brake.test.mjs`). The
occlusion culler carries the matching fault guard: a failed GPU query resolve logs
once and disables culling instead of storming unhandled rejections per frame
(`tools/occlusion-resolve-guard.test.mjs`). Device loss itself (VRAM exhaustion —
a bulk load + resize churn can OOM the device) is a queued-reload recovery:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tikimcfee/glyph3d-js](https://github.com/tikimcfee/glyph3d-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
