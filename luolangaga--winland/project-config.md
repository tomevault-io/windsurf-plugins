---
trigger: always_on
description: Windows Dynamic Island desktop app (WinUI 3 / Windows App SDK 2.3.1). Unpackaged WinExe — no MSIX, no store packaging. Projects: `WinIsland.Core/` (plugin SDK, v2.0), `WinIsland/` (main app), `samples/` (HelloPlugin / XamlPlugin / HardwareMonitor example plugins), `tools/pack-plugin.ps1` (`.lwp` packaging).
---

# AGENTS.md — WinIsland

## Project

Windows Dynamic Island desktop app (WinUI 3 / Windows App SDK 2.3.1). Unpackaged WinExe — no MSIX, no store packaging. Projects: `WinIsland.Core/` (plugin SDK, v2.0), `WinIsland/` (main app), `samples/` (HelloPlugin / XamlPlugin / HardwareMonitor example plugins), `tools/pack-plugin.ps1` (`.lwp` packaging).

Plugin authoring docs: `PLUGIN.md`.

## Build & Run

```powershell
cd WinIsland
dotnet build
dotnet run
```

No solution file (`.sln`) exists. Build from the `WinIsland/` directory directly.

Target: `net10.0-windows10.0.26100.0`, min `10.0.17763.0`. Requires .NET 10 SDK + Windows App SDK 2.3.1.

`AllowUnsafeBlocks` is enabled (used by Win32 P/Invoke in `Core/Win32.cs`).

## Architecture

```
WinIsland.Core/              — SDK DLL (luolan.winland.Core 2.2.1, distribute to plugin developers)
  IslandApi.cs               — ISettingsStore / IMorphView / IslandLiveContent / IslandMessage / IslandDropKind / IslandDropTarget / IslandDropContext / SettingsPageDescriptor
  IIslandPlugin.cs           — plugin entry point (InitializeAsync(IPluginContext) / ShutdownAsync)
  IPluginContext.cs          — IPluginContext / IIslandSurface / IPluginLogger
  IslandSdk.cs               — ApiVersion, HostVersion, manifest/package constants
  PluginManifest.cs          — manifest data type (plugin.json)
  IslandPluginBase.cs        — convenience base (Context/Log/Settings/SetContent/UpdateContent/RunOnUI)
  PluginXaml.cs              — supported way to load XAML from a plugin assembly (LoadComponent workaround)
  ActionDisposable.cs        — tiny IDisposable helper for plugin authors

WinIsland/                    — Main application
  App.xaml.cs                — Entry: IslandWindow, IslandService, PluginHost, built-in manifests, global crash logging
  Core/
    IslandService.cs         — host-side content/temp/settings-page API (RemoveSettingsPage, page Order)
    SpotlightHost.cs         — 「超级展开」聚光卡的生命周期：单实例/替换/关闭后放回岛体/通知旧 owner
    SettingsService.cs       — JSON key-value store at %LocalAppData%\WinIsland\settings.json
    Marketplace/             — plugin marketplace client: MarketplaceModels.cs (index.json schema 1) + MarketplaceService.cs
                               (single index.json request, ETag/TTL cache, mirror fallback, sha256-verified .lwp download)
    DropTargets/             — host-built-in file drop actions (open / reveal in Explorer / copy paths), registered
                               into IslandService from App.xaml.cs with Order 900+ so plugin targets sort first
    Update/                  — self-update check: UpdateModels.cs + UpdateService.cs (GitCode release API first, GitHub
                               fallback; update.autoCheck / update.lastCheck / update.skippedVersion; notify only —
                               it never downloads or installs anything, the browser does that)
    TransparentBackdrop.cs   — Fully transparent window backdrop (Composition + DWM alpha)
    TrayIcon.cs              — Native Shell_NotifyIcon tray icon with Win32 popup menu
    Win32.cs                 — All P/Invoke: window styles, DWM, subclassing for border removal, taskbar strip detection (`GetTaskbarStrip`), topmost-band self-heal (`EnsureTopmost`)
    TaskbarLayout.cs         — UI-Automation probe of the taskbar's *occupied* bands (so the island can be placed in a real free one); background-thread only, degrades to empty
    Plugins/                 — Plugin engine v2
      PluginHost.cs          — facade: discover/install/enable/disable/reload/uninstall, per-plugin serialization
      PluginInstance.cs      — state machine + timeouts + guarded callbacks + ALC unload & GC verification
      PluginAssemblyContext.cs — collectible ALC + AssemblyDependencyResolver + host-provided-assembly policy
      PluginScope.cs         — registration ledger; revoke = remove pages, clear content, dispose timers/subscriptions
      ScopedIslandSurface.cs / ScopedSettings.cs / PluginContext.cs — per-plugin scoped API surface
      PluginManifestReader.cs — plugin.json parsing + field-level validation
      LwpInstaller.cs        — .lwp (zip) validate/extract/atomic install/update/uninstall
      PluginLogService.cs    — per-plugin ring buffer + file logs (logs/plugin.<id>.log)
      PluginInfo.cs / PluginStateText.cs — runtime state shown in the plugin manager
  Island/
    IslandWindow.xaml.cs     — island window: state machine, size animation, hover, hit region, style switch, AnimateMorphView guard, position (top / bottom taskbar strip) + horizontal placement/offset, free-band placement, taskbar-follow poll, file-drop session (drag enter/over/leave/drop, edge auto-scroll, armed tile)
    DropStripView.cs         — 文件投放面板（拖文件进岛时岛体展开成的那一条）：左侧载荷摘要 + 右侧可横向滚动的投放卡片，卡片悬停放大/高亮全用组合级动画；只管长相与命中，会话状态在 IslandWindow
    MessageView.cs           — 临时消息卡（IslandMessage）的视图：图标芯片 + 标题 + 正文，宽度跟随内容、高度跟随行数；尺寸写死在视图上，过渡期间不重排

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [luolangaga/WinLand](https://github.com/luolangaga/WinLand) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
