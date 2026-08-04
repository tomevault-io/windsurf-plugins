---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture Rules

Before making any changes, read and follow all rules in `.claude/rules.md` and `.claude/project-overview.md`. These are non-negotiable and override any default behaviour.

---

## Build & Run

**Target:** .NET 10, x64 Windows only (`net10.0-windows`, `OutputType=WinExe`, `UseWindowsForms=true`).

```bash
# Build
dotnet build Yaesu_Web_Control.csproj

# Run (launches WinForms host + Kestrel on http://0.0.0.0:8080)
dotnet run --project Yaesu_Web_Control.csproj

# Publish self-contained
dotnet publish -c Release -r win-x64 --self-contained
```

There are no automated tests. Verification is manual via the browser at `http://localhost:8080`.

---

## Release Process

Before releasing: bump `Models/AppVersion.cs`, `installer.nsi`, add release notes to `README.md`, update `USER_MANUAL.md` if needed, and bump the per-release download badge in `README.md` to the new version (search for the previous `vX.Y.Z` tag in the shields.io URL near the top of the file).

```powershell
# 1. Commit everything on develop
git add -A
git commit -m "Release vX.Y.Z: ..."

# 2. Merge to main and tag
git checkout main
git merge develop --no-ff -m "Release vX.Y.Z"
git tag vX.Y.Z
git checkout develop

# 3. Push branches and tag
git push origin develop
git push origin main
git push origin vX.Y.Z

# 4. Create the GitHub Release — this triggers the build workflow
gh release create vX.Y.Z --title "vX.Y.Z" --notes "See README.md for full release notes."
```

**Step 4 is required.** The build workflow triggers on `release: [created]`, not on tag push alone.
Alternatively run `.\scripts\finish-release.ps1 -Version vX.Y.Z` which does all four steps.

User settings persist to `%APPDATA%\MM5AGM\Yaesu Web Control\appsettings.user.json`.  
Radio state persists to `%APPDATA%\MM5AGM\Yaesu Web Control\radio_state.json`.

---

## Backend Architecture

### Service Dependency Map

```
RadioInitializationService (IHostedService)
  └─ opens serial port via CatMultiplexerService
       └─ MultiplexedCatClient (ICatClient)
            └─ CatMessageDispatcher → RadioStateService → SignalR (RadioHub)
                                                        → RadioStatePersistenceService

MeterPollingService (IHostedService) — polls CAT FA/FB/SM etc. at ~10 Hz
SdrManager (IHostedService) — supervises one Yaesu_Sdr_Worker.exe per
                              configured SDR; reads FFT frames over
                              localhost TCP and broadcasts via SignalR
RigctldServer (IHostedService) — exposes rigctld TCP interface for WSJT-X etc.
```

### SignalR Message Envelope

All real-time updates use a single hub method `RadioStateUpdate` with envelope `{ property, value }`.  
The frontend's `WsUpdatePipeline` routes on `property`. The same hub carries:
- CAT state (FrequencyA, FrequencyB, PowerMeter, SMeter, etc.)
- SDR lifecycle (sdrId-tagged from v2.3.0):
  - `SdrStatus`     value = `{ sdrId, status }` — "unconfigured" / "connecting" / "streaming" / "disconnected" / "nodll"
  - `SdrError`      value = `{ sdrId, error  }` — human-readable detail
- SDR spectrum frames:
  - `SpectrumUpdate` value = `{ sdrId, bins, centreHz, spanHz }`

`sdrId` is `"A"` or `"B"`. `SdrSpectrumPipeline` routes by sdrId to the appropriate `SpectrumPanel` instance.

### CAT Frequency Format

`CatMessageDispatcher` parses `FA` / `FB` CAT responses. The FTdx101 sends frequencies as a plain integer string in **Hz** (e.g. `FA000880600;` = 880,600 Hz = 880.6 kHz). Values are stored and broadcast in Hz with no unit conversion. The FTdx101MP range is 30 kHz–75 MHz.

### Settings

`SettingsService` reads/writes `appsettings.user.json` via a read-modify-write pattern.  
`Settings.cshtml.cs`: `ModelState.Remove("Settings.SdrDeviceKeyA")` and `Settings.SdrDeviceKeyB` **must** appear before `ModelState.IsValid` — `<Nullable>enable</Nullable>` adds implicit `[Required]` to non-nullable strings, which silently blocks saves of empty values otherwise. The legacy `Settings.SdrDeviceKey` is also removed for the same reason; it's kept on the model as a migration anchor only.

**v2.2.x → v2.3.0 SDR settings migration:** the single `SdrDeviceKey` split into per-VFO `SdrDeviceKeyA` / `SdrDeviceKeyB`. `SettingsService.MigrateSdrDeviceKey` auto-promotes any legacy value into `SdrDeviceKeyA` on read; the legacy field is cleared on the next save.

### SDR Subsystem — Dual-process architecture (v2.3.0+)

The SDRplay API v3 service enforces one Selected device per host process (confirmed by [scripts/probe/](../scripts/probe/) — see [docs/decisions/0001-dual-sdr-architecture.md](../docs/decisions/0001-dual-sdr-architecture.md) for the four-pattern probe evidence). So:

- **YWC main never opens an SDR directly.**
- **`SdrManager`** (`Services/Sdr/SdrManager.cs`) spawns one `Yaesu_Sdr_Worker.exe` per configured device, connects to its localhost TCP port, reads FFT frames via `FrameReader`, and broadcasts them via SignalR with sdrId tagging.
- **`Yaesu_Sdr_Worker`** (`Workers/Yaesu_Sdr_Worker/`) — separate `.exe` per SDR. Each holds exactly one device. File-links the device code from `Services/Sdr/` so one source of truth.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mm5agm/Yaesu_Web_Control](https://github.com/mm5agm/Yaesu_Web_Control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
