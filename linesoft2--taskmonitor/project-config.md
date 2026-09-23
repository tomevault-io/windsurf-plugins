---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

**This file is a MAP, not the manual.** It carries the rules and the pointers; the long
unwindings live in `docs/` (loaded only when you touch that area) and at the code site.
See [Maintaining this file](#maintaining-this-file) before you add anything here.

| When you touch… | Read |
|---|---|
| taskbar geometry, boot/explorer-restart, D2D/DXGI, samplers, theme, process rows | `docs/gotchas.md` |
| a 设置 item's wiring to its sampler | `docs/settings-plumbing.md` |
| any of the six scroll views | `docs/scroll-stack.md` |

## What this is

A Windows taskbar widget: a self-drawn overlay embedded in the taskbar (three stacked groups — CPU/内存, 磁盘/GPU, 网络 ↑/↓) that pops up a fluent acrylic detail window on click. WPF on .NET Framework 4.8. **No main window**; the only persistent UI is the taskbar overlay. The process stays alive via `ShutdownMode="OnExplicitShutdown"`; exit is via the overlay's right-click menu.

## Build & run

```shell
dotnet build -c Debug          # only toolchain on this machine — no VS MSBuild / nuget.exe
bin\Debug\net48\task_monitor.exe
```

SDK-style csproj, `net48`, `UseWPF=true`. No tests.

**A live exe locks the output**, so `dotnet build` fails at the copy step with
`error MSB3021`/`MSB3027` naming `task_monitor.exe (PID)`. That is NOT a compile failure —
the compile succeeded. Exit the running instance first, and always use the sentinel:

```shell
touch bin/Debug/net48/shutdown.sentinel   # 1s tick notices it and exits gracefully
dotnet build -c Debug
bin/Debug/net48/task_monitor.exe &
```

The sentinel is the **default** path: this agent shell may be unelevated while the app is
elevated, and then `taskkill` fails with Access denied (UIPI) — file I/O is not gated that
way, which is exactly why the hook exists (mechanism: `ConsumeShutdownSentinel`,
TaskbarWindow.cs). Relaunching from an **unelevated** shell pops a UAC prompt on the secure
desktop — that one step is the user's; an elevated shell inherits elevation and skips it.

**The app always runs elevated, self-managed** (manifest `asInvoker`): `App.RunElevationGate` (src/App.xaml.cs) runs at the top of `OnStartup`, BEFORE the single-instance mutex — an exiting unelevated launcher must never hold the mutex, or the elevated child takes itself for a second instance. Unelevated + consented → `runas` self-relaunch; never asked → `ConsentDialog` (允许 persists `elevationConsent: true`); 不允许/UAC-cancel exits — the process never runs degraded (SRUM per-process net needs admin). Consent persists to **`settings.yaml` in the exe directory** (`AppSettings`, YamlDotNet — the single store for ALL settings; the file + `.tmp`/`.bad` siblings are runtime artifacts). The manifest's `dpiAware=true` keeps the overlay's D2D text sharp — don't drop it.

Exercising the consent dialog needs an unelevated launch (`explorer.exe "...task_monitor.exe"`); the UAC prompt lives on the secure desktop and cannot be automated. **Any verification needing a screenshot or visual confirmation is the user's job.**

Packages: `iNKORE.UI.WPF` + `iNKORE.UI.WPF.Modern` (control theming, merged in App.xaml), `DirectN` (overlay rendering), `FluentWpfCore` (popup acrylic + `SmoothScrollViewer`), `YamlDotNet`, framework `System.Drawing` (icon fallback only). Build-only: `Fody` + `Costura.Fody` — the build emits ONE exe (~6.1 MB), all managed DLLs woven in as compressed resources. **Costura, not ILRepack, deliberately** — ILRepack merges assemblies and would break the cross-assembly `pack://` URIs (rationale in task_monitor.csproj comments and the FodyWeavers.xml header).

**Single-instance:** a named `Global\` mutex right after the gate (`Global\TaskMonitor.exe__<guid>`, un-owned, existence test only; `Global\` so the guard holds across sessions — reachable only elevated, so `SeCreateGlobalPrivilege` is in hand). A second instance exits **silently** (the overlay is always visible anyway); a consented second launch still UAC-prompts first — inherent to the design.

**Legacy-OS warning:** right after the mutex, a pre-Win11 first launch pops `LegacyOsWarningDialog` once (`TaskbarWindow.IsWin11OrLater` — the raw OS check, no taskbar-shape test) saying Win10 compatibility issues are expected and won't be fixed; `legacyOsWarningShown: true` in settings.yaml suppresses it thereafter. **Win10 support is deprioritized** — the classical taskbar path below still ships, but new work targets Win11 only.

## 发布 / CI/CD

**GitHub 是主仓库，CNB 是镜像。** Remotes: `origin` = GitHub（SSH，`git@github.com:linesoft2/TaskMonitor.git`），`cnb` = CNB（HTTPS——CNB **不支持 SSH**（官方明示），HTTPS git 认证 = 固定用户名 `cnb` + 访问令牌）。日常提交只推 `origin`；两个 GitHub Actions workflow（.github/workflows/）负责同步和发布：

- `sync-cnb.yml`（main 分支 + v* tag 推送）→ `docker://tencentcom/git-sync` 把代码和 tag **force** 推到 CNB（CNB 侧是纯镜像、从无独有提交，force 保证一致）。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [linesoft2/TaskMonitor](https://github.com/linesoft2/TaskMonitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
