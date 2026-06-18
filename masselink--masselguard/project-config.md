---
trigger: always_on
description: WireGuard tunnel manager for Windows. Two executables:
---

# MasselGUARD — Codebase Guide

WireGuard tunnel manager for Windows. Two executables:
- **`MasselGUARD.exe`** — WPF GUI application (`OutputType=WinExe`, no console flash)
- **`MasselGUARDcli.exe`** — Console CLI (`OutputType=Exe`, PowerShell/cmd wait for exit)

.NET 10.

## Build

```bat
BUILD.bat          # requires .NET 10 SDK; output → dist\
```

Produces both `dist\MasselGUARD.exe` and `dist\MasselGUARDcli.exe`.

Current version: **3.6.0 — Dangerous Donkey**
When bumping version, update **both** `UpdateChecker.cs` (`CurrentVersion` + `_codenames`) **and** `BUILD.bat` (`VERSION` + `CODENAME`).

## Key design decisions

- **Two exe split** — `MasselGUARD.exe` is `WinExe` so Windows never allocates a console (no flash). `MasselGUARDcli.exe` is `Exe` so terminals wait for it. Source is shared via `<Compile Include>` links from `MasselGUARDcli/MasselGUARDcli.csproj`.
- **`GenerateAssemblyInfo=false` + `GenerateTargetFrameworkAttribute=false`** on `MasselGUARD.csproj` — suppresses duplicate attributes caused by WPF's wpftmp compile step when `OutputType=WinExe`. Version info is still embedded via the wpftmp project's own generated attributes using `-p:` overrides from BUILD.bat.
- **`requireAdministrator` manifest** — UAC always elevates. Non-admin terminals get an isolated console (new window). `IsIsolatedConsole()` via `GetConsoleProcessList` detects this and pauses before exit.
- **`TearDownAdapter`** — after `EnsureStopped`, calls `WireGuardOpenAdapter` + `WireGuardCloseAdapter` to release any lingering kernel adapter. Needed because the WireGuardTunnelService exits but the adapter can outlive it.
- **`IsRunning()`** — checks `ServiceController.Status == Running` first (primary), then the WireGuard pipe as fallback.
- **`RefreshStatus()` resets DNS badge** — when poll detects tunnel going inactive externally (CLI disconnect), resets `_dnsStatus` and fires PropertyChanged for all DNS properties.
- **`KillSwitchService.Disable()`** — early-returns if tunnel not in `_active` HashSet, preventing spurious `[KillSwitch] Disabled` log entries.
- **Auto-reconnect** — `TunnelService._intentionalDisconnects` (ConcurrentDictionary) is populated at the top of every `Disconnect()` call. `MainViewModel.IsIntentionalDrop()` consumes it via `ConsumeIntentionalDisconnect()` before triggering `AutoReconnectAsync()`. This ensures WiFi-rule, CLI, and user disconnects are never retried. `AutoReconnectAsync` retries up to 3 times with 5 s / 10 s / 15 s backoff. Global mode (`AppConfig.AutoReconnectMode`) + per-tunnel flag (`StoredTunnel.AutoReconnect`) resolved by `TunnelService.ShouldAutoReconnect()` — same pattern as kill switch. Because the poll never sees MasselGUARD's own transitions (`DoDisconnect` refreshes `IsActive` immediately), an intentional mark would otherwise go stale — so a successful `Connect()` removes it again. The WireGuard app's deactivate stops the service *before* deleting its SCM entry, so the crash-vs-deactivate check (`WireGuardServiceExists`) at drop time races the deletion; `AutoReconnectAsync` therefore starts with a 2 s grace check that recognises a clean deactivate before announcing any reconnect countdown, and re-checks after each backoff delay in case the user deactivates mid-loop. Reconnect attempts `await vm.ConnectAsync()` (awaitable extraction of `DoConnect`) — firing `ConnectCommand` and reading `IsActive` immediately reported every attempt as failed.
- **External companion connect/disconnect** — when the 1 s poll sees a companion tunnel transition that MasselGUARD didn't initiate (WireGuard app / CLI), `MainViewModel.RefreshTunnelStatus` calls `TunnelService.RecordExternalConnect` (opens history entry, snapshots byte counters, clears stale intentional mark + `UserDisconnected`) or `RecordExternalDisconnect` (closes the open history entry via `LogDisconnect`, which also writes the `Disconnected: <name>` log line). Without this, externally dropped tunnels left history entries open forever and never appeared in the activity log.
- **`UpdateChecker.UpdateAsync`** — takes an `onShutdown` callback so WPF-specific `Application.Current.Dispatcher.Invoke(ShutdownApp)` stays in the GUI call site, keeping `UpdateChecker.cs` WPF-free.
- **File-only tunnel config storage** — `StoredTunnel.Config` (inline DPAPI blob) is legacy. All new saves write a `.conf.dpapi` file to `%APPDATA%\MasselGUARD\tunnels\` and store only the `Path`. `ConfigService.Load()` migrates old inline entries to files automatically on first run and nulls `Config` so it disappears from `config.json`. The `[JsonIgnore(Condition = WhenWritingNull)]` attribute ensures `Config` is never written once cleared.
- **`TunnelService.SaveConfigToFile`** — DPAPI-encrypts plaintext and writes to `TunnelStorageDir`. Used by GUI add/edit, import dialog, QR import, and CLI import/rawconnect. Returns the file path stored in `StoredTunnel.Path`.
- **`ApplyWifiState`** — central method called from both `OnWifiChanged` (WLAN notification) and `TryUpdateWifi` (startup query). Records SSID history and updates all UI. Ensures the current SSID is captured immediately at startup rather than only on WiFi changes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [masselink/MasselGUARD](https://github.com/masselink/MasselGUARD) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
