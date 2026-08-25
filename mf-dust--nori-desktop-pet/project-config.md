---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository layout

Two independent deliverables, no shared build:

- `app/desktop/` — the Nori desktop pet. **.NET 10 + Avalonia 12 host** (`Nori.Desktop/`, `Nori.Core/`, C#) + Vue 3 SPA (`src/`, TypeScript + **UnoCSS**) rendered in Avalonia's **cross-platform NativeWebView** (WebView2 on Windows, WKWebView on macOS, WebKitGTK on Linux). This is where nearly all work happens.
- `docs/` — Chinese design docs. `规范.md` is a binding style contract, not advice — read it before touching frontend or C# code. `技术.md` is the module/tech map (and records the pet-window transparency verification), `跨平台.md` the platform support matrix + degradation table, `开发任务清单.md` the roadmap, `windows.md` an Avalonia window-property reference.

`README.md` is empty.

## Commands

Desktop app — run from `app/desktop/`. **Use pnpm**: the project is managed with pnpm, and `node_modules` layout assumptions in scripts assume it.

```bash
pnpm install          # 安装前端依赖
pnpm build            # vue-tsc --noEmit && vite build  ← the frontend gate
pnpm test             # vitest run  ← 前端纯函数/服务回归
dotnet build          # builds Nori.Core + Nori.Desktop + tests  ← the backend gate
dotnet test           # xUnit; pure-function coverage
./publish.bat         # framework-dependent publish (no bundled runtime) for win-x64
./publish.sh          # same, for linux-x64 / linux-arm64 / osx-arm64 / osx-x64
```

CI (`.github/workflows/build.yml`) runs all four gates on `windows-latest` / `ubuntu-latest` / `macos-14` plus a publish smoke test.

Running the app:

```bash
dotnet run --project Nori.Desktop            # production: serves the built dist/ from wwwroot
NORI_DEV=1 dotnet run --project Nori.Desktop # dev: points the WebView at vite on :1420
pnpm dev                                      # vite only; must be running for NORI_DEV=1
```

`规范.md` requires `pnpm build`, `dotnet build` and `dotnet test` to pass before a change is considered done. `tsconfig.json` sets `noUnusedLocals`/`noUnusedParameters`; the C# projects set `TreatWarningsAsErrors`.

## Architecture

### Four windows, three NativeWebView + one native OpenGL

`Nori.Desktop/Windows/WindowDefinition.cs` declares four windows — `first-run`, `init`, `main`, `pet` — all created hidden, borderless (`WindowDecorations.None`) and transparent.
- Three windows (`first-run`, `init`, `main`) are `NoriWindow` hosting `NativeWebView` and loading the Vue bundle. `main` doubles as the **audio host** (see below) — it only hides on close, so it is always alive.
- The desktop pet (`pet`) is a native Avalonia `PetWindow` hosting `PetGlControl` (OpenGL via `Live2DCSharpSDK`), bypassing webview airspace and window-region clipping issues completely. Same code on all three desktops.

1. `App.cs` reads `first_run_completed` from SQLite and shows `first-run` or `init`.
2. The host navigates each webview to `…/app/index.html?window=<label>`.
3. Each webview mounts `App.vue`, which calls `navigateToOwnWindow()` — it reads its own label from the query string and `router.replace()`s to the mapped route.
4. The label→route table is `WINDOW_ROUTES` in `src/services/window/index.ts`.

**Adding a webview window means touching four places**: `WindowDefinition.All`, the `WindowLabel` union, `WINDOW_ROUTES`, and `src/services/router/index.ts`.

First-run flow: wizard in `first-run` → `complete_first_run` (C#) sets an `initStartPending` flag, closes `first-run`, shows `init`, broadcasts `nori:init-start` → `InitView` opens `main`, then closes `init`.

**The broadcast can outrun the subscription.** `init` starts hidden on the first-run path, and its webview may still be loading when `nori:init-start` fires — the event is then lost forever and the page spins on the loading ring. So `InitView` **subscribes first, then calls `init_ready`**, which returns (and clears) `initStartPending`; a true value means "run the flow now". `startInitFlow` has a reentrancy gate, and a 10s watchdog offers a manual "open main window" escape hatch.

The tray (`Tray/TrayMenu.cs`) is the primary always-available entry point: left-click opens `main`, menu toggles `pet`. `Install` returns `false` when the tray fails (some Linux desktops have no StatusNotifier), which flips `platform.supportsTray` so the frontend shows a built-in entry instead. Closing a window only hides it (`NoriWindow.AllowClose` / `PetWindow.AllowClose` gates real disposal); `ShutdownMode.OnExplicitShutdown` keeps the process alive.

`WindowManager` tracks each window's `IsVisible` and raises `VisibilityChanged`; `AppRuntime` turns that into `InvalidateSnapshot("pet")`, so `snapshot.pet.visible` is the single source of truth for pet state — toggling from the tray updates the main window immediately.

### The bridge (replaces Tauri IPC)

`NativeWebView` only offers JS→host `invokeCSharpAction(string)` and host→JS `InvokeScript`. On top of that:

- **Bootstrap** — an inline `<script>` in `index.html`, before the module script, defines `window.__nori` (`invoke` / `emit` / `listen` / `dispatch`, plus `label` and `assetBase`). It must stay synchronous and first.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MF-Dust/Nori-Desktop-Pet](https://github.com/MF-Dust/Nori-Desktop-Pet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
