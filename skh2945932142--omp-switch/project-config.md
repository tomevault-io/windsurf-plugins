---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

OMP Switch is a Windows-first Electron desktop app that manages [Oh My Pi](https://github.com/can1357/oh-my-pi) (OMP) model-provider configuration. It edits **user-owned files it does not own**: `~/.omp/agent/models.yml` and `config.yml`. Everything about the architecture follows from that: hash-guarded writes, YAML comment preservation, snapshots before every commit, and read-only mode for unknown OMP schema versions.

## Commands

Requires Windows, Node 24+, pnpm 11 (`corepack enable`), **.NET SDK 10.0**, and the Visual Studio **"Desktop development with C++"** workload — `native/secret-bridge` publishes as Native AOT, which links with MSVC. `scripts/build-secret-bridge.ps1` prepends the fixed `vswhere.exe` location to PATH because the ILCompiler shells out to it and Visual Studio does not put it on PATH; without that a machine with the C++ workload still fails to link with a confusing `MSB3073`.

```powershell
pnpm install --frozen-lockfile
pnpm dev                 # predev runs build:native first, so dotnet is required even for dev
pnpm typecheck           # tsc --noEmit over electron/, src/, packages/
pnpm test                # vitest run
pnpm test:watch
pnpm build               # build:native + electron-vite build -> out/
pnpm package:win         # -> dist/ NSIS installer + portable ZIP
pnpm verify:package-cli  # runs the packaged JSON CLI in a temp HOME; needs dist/ from package:win
pnpm build:native        # build:secret-bridge + build:cli-proxy (dotnet publish)
```

Single test file / single case:

```powershell
pnpm vitest run packages/core/src/gateway.test.ts
pnpm vitest run packages/core/src/gateway.test.ts -t "fails over"
```

CI (`.github/workflows/ci.yml`) runs typecheck → test → build on `windows-latest`, then **fails if the build left any tracked or unignored file**. New build artifacts must be added to `.gitignore`.

### Running against a throwaway HOME

The adapter takes its profile root from `os.homedir()` with no override, so `pnpm dev`, `--json`, and `--gateway` all read and write the **developer's real `~/.omp/agent/models.yml`**. Before exercising a write path manually, redirect both the profile root and the app data dir — this is exactly what `scripts/verify-packaged-cli.ps1` does:

```powershell
$env:USERPROFILE = "D:\tmp\omp-home"; $env:HOME = $env:USERPROFILE  # moves ~/.omp (os.homedir())
$env:OMP_SWITCH_DATA_DIR = "D:\tmp\omp-data"                        # moves userData: vault, snapshots, metadata
```

`OMP_SWITCH_DATA_DIR` is honored by `electron/main.ts` (`app.setPath("userData", …)`) *and* independently by the C# secret bridge, which otherwise defaults to `%APPDATA%\omp-switch` — keep the two in agreement or key lookups break. Other env inputs: `ELECTRON_RENDERER_URL` (set by electron-vite in dev; absent means load the built `index.html`) and `OMP_AUTH_GATEWAY_URL` (gateway upstreams of kind `omp-auth-gateway`, default `http://127.0.0.1:4000`).

## Architecture

Three layers with a strict dependency direction: `packages/core` → `electron/` → `src/renderer`.

**`packages/core/src` — all domain logic, zero Electron imports.** Pure Node + TypeScript so it is directly unit-testable. Imported everywhere as `@omp-switch/core`, an alias (not a built package) declared in three places that must stay in sync: `tsconfig.json` paths, `electron.vite.config.ts` (main + renderer), and `vitest.config.ts`. New modules must be re-exported from `packages/core/src/index.ts`.

**`electron/` — main process.** Owns the OS: `ipcMain.handle` surface in `main.ts`, `safeStorage` credential vault (`secret-store.ts`), and `metadata-store.ts`. Holds no domain logic of its own. `createWindow` enables the Mica material on Windows 11 22H2+ (`backgroundMaterial: "mica"`, mutually exclusive with an opaque `backgroundColor`) and injects the `mica` class on `<html>` via `executeJavaScript` after load; `tokens.css` makes only the chrome transparent under that class, panels stay opaque, and every other environment falls back to solid surfaces. On Windows 10+ it also hides the OS title bar (`titleBarStyle: "hidden"` + `titleBarOverlay`) so the web topbar is the drag region — `.topbar` carries `-webkit-app-region: drag` with buttons opted back out, `.topbar-actions` reserves right padding for the overlay buttons, and a `nativeTheme.on("updated")` listener re-tints the overlay glyphs. `app:set-theme` forwards the renderer's manual theme choice into `nativeTheme.themeSource` so those glyphs follow it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skh2945932142/omp-switch](https://github.com/skh2945932142/omp-switch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
