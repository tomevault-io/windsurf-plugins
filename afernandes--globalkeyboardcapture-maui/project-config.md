---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Layout

Two projects in one solution (`GlobalKeyboardCapture.Maui.sln`):

- `GlobalKeyboardCapture.Maui/` — the NuGet library. Multi-targets `net8.0-android;net9.0-android` and (only on Windows hosts) `net8.0-windows10.0.19041.0;net9.0-windows10.0.19041.0`.
- `GlobalKeyboardCapture.Maui.Sample/` — the consumer app used to manually exercise the library. Targets `net9.0-android` (+ `net9.0-windows...` on Windows).

The library version is the `<CurrentVersion>` property in `GlobalKeyboardCapture.Maui.csproj` (currently `1.0.2`); the NuGet package is built only on `Release` (`GeneratePackageOnBuild=true`).

## Build / Run Commands

```bash
# Restore and build the whole solution
dotnet restore GlobalKeyboardCapture.Maui.sln
dotnet build   GlobalKeyboardCapture.Maui.sln

# Build a specific target framework (Windows targets only build on Windows hosts)
dotnet build GlobalKeyboardCapture.Maui/GlobalKeyboardCapture.Maui.csproj -f net9.0-android
dotnet build GlobalKeyboardCapture.Maui/GlobalKeyboardCapture.Maui.csproj -f net9.0-windows10.0.19041.0

# Pack the NuGet (uses Release config; produces .nupkg + .snupkg)
dotnet pack  GlobalKeyboardCapture.Maui/GlobalKeyboardCapture.Maui.csproj -c Release

# Run / deploy the sample
dotnet build GlobalKeyboardCapture.Maui.Sample/GlobalKeyboardCapture.Maui.Sample.csproj -t:Run -f net9.0-windows10.0.19041.0
```

There is **no test project and no lint task** in this repo. Style is enforced via `.editorconfig` (4-space indent, `utf-8-bom`, system usings sorted first). When stale build artifacts cause weird MAUI/Android errors, run the PowerShell helper `delete-bin-obj.ps1` to wipe every `bin/`, `obj/`, and `.vs/`.

## Architecture

The library is a small **dispatcher + handler pipeline** that sits on top of a per-platform key-event source.

```
Platform OS event
  → IPlatformKeyHandler  (Windows: PreviewKeyDown / Android: Window.Callback)
  → KeyEventArgs (unified model)
  → KeyHandlerService.HandleKeyPress
  → foreach IKeyHandler: ShouldHandle(key) ? HandleKey(key)
```

### Wiring (two-step opt-in by the consumer)

Both calls are required in `MauiProgram.cs`:

1. `builder.UseKeyboardHandling()` (`Configuration/MauiAppBuilderExtensions.cs`) — registers MAUI lifecycle hooks. On Windows it fires on `OnLaunched`; on Android on `OnCreate`/`OnResume`. The hook resolves `ILifecycleHandler` and calls `OnStart`/`OnResume`.
2. `builder.Services.AddKeyboardHandling(options => …)` (`Configuration/ServiceCollectionExtensions.cs`) — DI registration. `IKeyHandlerService`, `ILifecycleHandler`, and `IPlatformKeyHandler` are singletons; `BarcodeHandler` and `HotkeyHandler` are **transient** (each page gets its own instance).

The lifecycle handler (`KeyHandlerLifecycleHandler`) is what actually grabs the platform "view" and calls `IKeyHandlerService.Initialize(...)`:
- Windows: `Application.Current.Windows[0].Handler.PlatformView` cast to `Microsoft.UI.Xaml.Window`.
- Android: `Platform.CurrentActivity.Window.DecorView.RootView`.

This is why a consumer page only needs to `RegisterHandler(...)` — the platform hook-up has already happened.

### `KeyEventArgs` and the hotkey string contract

`Core/Models/KeyEventArgs.cs` is the single normalized event shape produced on both platforms. Its `ToString()` is **the lookup key** for `HotkeyHandler` — modifiers always emitted in the order `Ctrl+Alt+Shift+Win`, then a function key / character / special key name (`Enter`, `Esc`, `Up`, `Space`, …). When neither `Character` nor a named key was set, it falls back to `OEM{VirtualKey}` on Windows or the Android `KeyCode` name. **Any change to `ToString()` is a breaking change for registered hotkeys.**

`HotkeyHandler.NormalizeHotkey(...)` mirrors the same modifier order and accepts aliases (`Control`→`Ctrl`, `Windows`→`Win`, `ESCAPE`→`Esc`). Registration is case-insensitive and order-insensitive — keep these two normalizers in sync.

### Platform handlers

- `Platforms/Windows/WindowsKeyHandler.cs` — subscribes to `PreviewKeyDown` on the window's content; reads modifier state via `InputKeyboardSource.GetKeyStateForCurrentThread`. `KeyboardHelper` maps `VirtualKey` → `char` (letters, numerics, numpad, OEM punctuation 186–222) and `F1`–`F24`.
- `Platforms/Android/AndroidKeyHandler.cs` — installs a `Window.ICallback` proxy (`KeyEventCallback`) that forwards every `DispatchKeyEvent`. Only events with `Action == Down && Flags == FromSystem` are processed; the rest pass straight through to the original callback. **Always preserve and forward to the original `IWindowCallback`** — replacing it without delegation will break MAUI's input pipeline. Cleanup must restore `_originalDispatcher`.

`KeyboardHelper` exists in **both** platform folders with the same public surface (`ToChar`, `ToFunction`) but different inputs — Windows works from `VirtualKey`, Android from `KeyEvent.DisplayLabel` strings. Don't try to share them.

### Adding a new `IKeyHandler`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [afernandes/GlobalKeyboardCapture.Maui](https://github.com/afernandes/GlobalKeyboardCapture.Maui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
