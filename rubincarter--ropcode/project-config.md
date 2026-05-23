---
trigger: always_on
description: This file gives Codex and other coding agents the repository-specific context needed to work safely in this project.
---

# AGENTS.md

This file gives Codex and other coding agents the repository-specific context needed to work safely in this project.

## Project Overview

Ropcode is an Electron desktop app that wraps Claude Code, Gemini CLI, and Codex so users can run AI coding agents in parallel. The repository builds three runtime surfaces from one Go module:

- `ropcode-server`: WebSocket RPC backend, built with `-tags server` from `server_main.go`.
- `ropcode`: CLI binary under `cmd/ropcode`, connecting to a running server over RPC.
- Electron shell: `electron/` starts `ropcode-server` and serves the React frontend.

The standalone server mode is valid. In development it reverse-proxies Vite; in production it serves `frontend/dist`.

## Common Commands

Run these from the repository root unless noted.

| Purpose | Command |
| --- | --- |
| Full dev stack (Go + frontend + Electron) | `npm run dev` |
| Electron-only dev (Go + frontend already built) | `make dev` |
| Full production build | `npm run build` |
| Electron-only build | `make build` |
| Packaged Electron release | `npm run build:release` |
| Wails single-exe build | `.\scripts\build-wails.ps1` |
| Go server and CLI build | `npm run build:go` |
| Go CLI dev build | `npm run build:cli:dev` |
| Go tests | `go test ./...` |
| Single Go test | `go test -run TestName ./path/to/pkg` |
| Electron tests | `cd electron && npm test` |
| Frontend typecheck | `cd frontend && npm run build:typecheck` |
| Clean | `make clean` |

The `Makefile` is a thin shim — its `dev` and `build` targets only run `cd electron && npm run dev|build`. They do not rebuild the Go binaries or start Vite. Use the `npm run dev` / `npm run build` scripts when you need the full stack.

Build note: a plain `go build .` does not produce a runnable server because `server_main.go` is behind the `server` build tag. Use `go build -tags server -o bin/ropcode-server .` or the npm scripts.

Wails build note: do not ship or test `build-wails\bin\RopcodeWails.exe` produced by bare `go build -tags wails`. It can compile but will fail at startup with Wails' "correct build tags" dialog because the Wails CLI adds required production tags, metadata, and packaging steps. Use `.\scripts\build-wails.ps1` for a runnable single-exe Wails build.

## Architecture Notes

### Electron to Go Startup

`electron/src/go-server.ts` starts `ropcode-server` with:

- `ROPCODE_AUTH_KEY`
- `ROPCODE_MODE=websocket`
- `ROPCODE_VITE_URL` in dev or `ROPCODE_FRONTEND_DIR` in production

The Go server prints `WS_PORT:<port>` to stdout. Electron parses that line and loads the UI through the Go server. The browser should talk to Go, not directly to Vite.

When testing the unpacked packaged app from `release\win-unpacked\Ropcode.exe`, Electron does not use root `bin\` or `frontend\dist` directly. It starts:

- `release\win-unpacked\resources\bin\ropcode-server.exe`
- `release\win-unpacked\resources\frontend`

After changing backend or frontend code for a packaged-app repro, rebuild and copy the updated artifacts into those `resources` paths, or run a full release packaging step. Verify with `Get-Process ropcode-server | Select Path`, `Get-FileHash`, or a fresh server log marker before judging the fix.

For Windows release work, `npm run build:release` may fail if it is launched through `cmd.exe`, because it invokes `./scripts/build-electron.sh`. Run it from Git Bash or call `bash ./scripts/build-electron.sh` directly.

For a single-file Windows test build, use the portable target instead of NSIS. The portable output is `release\Ropcode 0.x.x.exe`; it still relies on the unpacked `resources` tree at runtime, so verify both the exe and `release\win-unpacked\resources\bin\ropcode-server.exe` hash after packaging. The current portable config needs Windows-specific `extraResources` paths (`bin/win32/x64/...`) rather than the default macOS placeholders in `electron-builder.yml`.

### Wails Single-Exe Shell

The Wails shell is an additive Windows build path configured by `wails.json` and `scripts/build-wails.ps1`. It embeds `frontend/dist` into `build-wails\bin\RopcodeWails.exe`, starts the existing `BootstrapRuntime` in-process, and exposes the same WebSocket RPC routes through the Wails asset server.

For Wails repros or distribution tests:

- Build with `.\scripts\build-wails.ps1`; add `-SkipFrontend` only when `frontend/dist` has already been rebuilt from the current frontend source.
- Do not replace this with `go build -tags wails`; that omits required Wails production build tags and produces an exe that opens an error dialog.
- After frontend or backend changes, verify the running process path is `build-wails\bin\RopcodeWails.exe` and check its `LastWriteTime` or hash before judging the fix.
- Wails uses the system WebView2 runtime and does not bundle Electron, Chromium, Bun, or Node. The Electron `<webview>` feature is not full parity in this shell.

### Reflection RPC

`internal/websocket/router.go` reflects exported methods on `*App` and exposes them as RPC endpoints. To add a frontend-callable API:

1. Add a public method on `*App`, usually in `bindings.go` or another file in package `main`.
2. Add a typed wrapper in `frontend/src/lib/rpc-client.ts`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RubinCarter/ropcode](https://github.com/RubinCarter/ropcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
