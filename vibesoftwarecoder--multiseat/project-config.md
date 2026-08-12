---
trigger: always_on
description: MultiSeat runs multiple simultaneous Moonlight game-streaming sessions on one Windows host. Each seat gets an isolated Windows account, virtual display (SudoVDA), virtual audio device, and a dedicated Apollo streaming instance, managed from a web dashboard.
---

# MultiSeat — Codebase Guide

MultiSeat runs multiple simultaneous Moonlight game-streaming sessions on one Windows host. Each seat gets an isolated Windows account, virtual display (SudoVDA), virtual audio device, and a dedicated Apollo streaming instance, managed from a web dashboard.

## Stack

- **Backend:** .NET 9 / ASP.NET Core Windows Service (`src/MultiSeat.Service`) — runs as SYSTEM
- **Frontend:** React + TypeScript dashboard (`src/MultiSeat.Dashboard`) — Vite build, served on port 9550
- **Shared:** `src/MultiSeat.Shared` — constants, port layout, shared types
- **Tests:** `src/MultiSeat.Tests`
- **InputHook DLL:** `src/MultiSeat.InputHook` — C++/CMake, keyboard/mouse session isolation
- **Solution:** `src/MultiSeat.slnx`

## Key Service Components

| Class | Responsibility |
|---|---|
| `SeatManager` | Seat lifecycle — provision / teardown |
| `SessionLauncher` | RDP loopback session creation, mstsc window management |
| `ApolloManager` | Per-seat Apollo process management |
| `VirtualDisplayManager` | SudoVDA virtual display attach/detach |
| `AudioRouter` | Virtual audio device assignment per seat |
| `InputRouter` | XInput/ViGEm controller routing |
| `HidHideConfigurator` | Controller cloaking via HidHide |
| `InputHookManager` | Keyboard/mouse session isolation (InputHook DLL) |
| `AccountManager` | Windows local account CRUD |
| `ApiServer` | ASP.NET Core HTTP API + WebSocket |

## Port Layout

Each seat reserves a block of 30 ports: `PortBase + (seat_index × 30)`. Default `PortBase = 48100`.
- Seat 0 → 48100–48129, Seat 1 → 48130–48159, etc.
- Apollo's per-seat offsets span `-5` (GFE HTTPS) to `+26` (RTSP) around the base.
- The base sits **above** a stock Apollo's block (~47979–48010, centered on the Moonlight default 47984) so MultiSeat coexists with a standalone Apollo — see "Coexistence with a standalone Apollo" below.

## Audio Device Layout

- Seat 0 → VB-CABLE basic "CABLE Input"
- Seat 1 → VoiceMeeter "VoiceMeeter Input"
- Seat 2 → VoiceMeeter "VoiceMeeter Aux Input"
- Seat 3 → VoiceMeeter "VoiceMeeter VAIO3 Input"

VoiceMeeter must be running — `AudioRouter` auto-starts it. Registered in `HKLM\Run` for auto-start at boot.

## Streaming behavior — resolution, audio, controller

Reflects fixes shipped 2026-07-24 (GitHub issues #11 / #10 / #9a):

- **Resolution follows the Moonlight client.** Each per-seat `sunshine.conf` sets Apollo's display-device keys — `dd_configuration_option = ensure_active`, `dd_resolution_option = auto`, `dd_refresh_rate_option = auto` (`ApolloConfigBuilder`). Apollo resizes the SudoVDA display to the mode the client requests on connect. The dashboard resolution is the SudoVDA creation/advertised default, **not** authoritative. **Requires the client's "Optimize game settings" (SOPS) enabled** — otherwise Apollo leaves the mode unchanged. Without the `dd_*` keys, `dd_configuration_option` defaults to `disabled` and Apollo never resizes the virtual display (it stays at the host/RDP surface size).
- **Seat audio does not hijack the host default.** The seat's virtual audio device is written as Apollo's `virtual_sink` (not `audio_sink`) with `keep_sink_default = disabled` + `auto_capture_sink = disabled`, and MultiSeat no longer runs `--set-default-render`. Windows has a **single machine-wide default output** shared by the console + all seats; Apollo still points the game at the sink during an active stream and restores the previous default afterward, without re-asserting it. Keep the client's "Play audio on host PC" **off** so `virtual_sink` is used. Not full isolation — true N-way seat audio needs per-app routing (`IAudioPolicyConfigFactory::SetPersistedDefaultAudioEndpoint`), not yet built.
- **Controller forwarding is native by default.** `EnableViGEmController` defaults **off** — Apollo forwards the Moonlight client's controller into the seat itself and MultiSeat creates no ViGEm pad. The dashboard shows the seat's Controller service as **"Native"** (not a down light) and the Input tab notes that XInput→seat assignment only applies when `EnableViGEmController` is on. `SeatServices.ControllerManaged` + `GET /api/input/mode` surface the mode.

## Install / Deploy

Two separate scripts — prereqs and service deploy are intentionally split:

```powershell
# Step 1: Install all prerequisites (drivers, audio devices, RDPWrap, etc.)
.\prerequisites\install-prerequisites.ps1
# Reboot if prompted, then re-run to confirm clean.
# Log: prerequisites\prereq.log

# Step 2: Build and deploy the MultiSeat service (run from scripts\)
.\scripts\install-service.ps1

# Remove the service
.\scripts\install-service.ps1 -Uninstall
```

## Key Runtime Paths

| Path | Purpose |
|---|---|
| `C:\Program Files\MultiSeat\` | Service install dir |
| `C:\Program Files\ApolloVibe\` | MultiSeat's own Apollo install (separate from any standalone `C:\Program Files\Apollo\`) |
| `C:\ProgramData\MultiSeat\` | Runtime data, Apollo configs, logs |
| `C:\ProgramData\MultiSeat\logs\` | `audio-helper.log` only — **not** the service log (see below) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vibesoftwarecoder/MultiSeat](https://github.com/vibesoftwarecoder/MultiSeat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
