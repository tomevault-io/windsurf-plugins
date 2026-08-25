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

`README.md` contains the project overview, current stabilization boundary and development gates.

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

### Stabilization contracts

- 产品版本由构建环境控制；未显式注入时唯一默认值是 `Dev`。Release 必须手动输入唯一 codename，并通过 `NORI_PRODUCT_VERSION` 注入稳定版本、通过 `NORI_PRODUCT_INFORMATIONAL_VERSION` 注入带 `NORI_COMMIT_SHA` 短 hash 的 informational version。CLR/File/NuGet 版本保持数字格式，`ProductVersion.Current` 保留完整 `v<version>-<codename>+<shortsha>` informational version。不要新增 `0.1.0` 回退。`ProductVersion.Current` 进入 snapshot、Smoke readiness、诊断、Crash 报告和 MCP clientInfo。
- `--safe-mode` 只接受人工命令行启动，不自动恢复。它保留 UI、日志、诊断和本地手动修复，跳过 MCP 自动连接、Proactive/Reflection、知识与记忆后台维护、AI 桌宠交互及 Live2D 自动模型加载；Bridge 入口同时拒绝交互式联网、Provider/MCP/语音外部操作。
- `readiness.json` schema v2 必须包含 `product_version`、`database_schema_version`、`config_schema_version` 和 `safe_mode`；Windows CI 覆盖 first-run、initialized 和 initialized safe-mode。
- `export_diagnostics` 只生成大小受限、脱敏白名单 ZIP；不得包含数据库、聊天/记忆/提示词、工具参数/结果、请求正文、录音、资源、凭据或真实用户路径。Provider 连接测试发送固定探测且不持久化配置或内容。
- 迁移前备份使用 `VACUUM INTO`，单文件上限 64 MiB、最多保留 3 份；新增迁移必须保持这条保护。

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


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MF-Dust/Nori.Desktop](https://github.com/MF-Dust/Nori.Desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
