---
trigger: always_on
description: C++20 Windows screen-mirroring receiver (AirPlay, Android/scrcpy, Miracast, Cast).
---

# AGENTS.md — 1PhoneMirror

C++20 Windows screen-mirroring receiver (AirPlay, Android/scrcpy, Miracast, Cast).
Single-binary SDL2 + FFmpeg desktop app, plus a small C# Azure Functions
telemetry backend under [telemetry/](telemetry/).

For user-facing docs see [README.md](README.md), [GETTING_STARTED.md](GETTING_STARTED.md),
[PRIVACY.md](PRIVACY.md), [GOVERNANCE.md](GOVERNANCE.md), [DISTRIBUTION.md](DISTRIBUTION.md).

## Build & run

```powershell
# First time only — installs vcpkg, FFmpeg, SDL2, OpenSSL
.\scripts\setup_deps.ps1
# Every build (Release by default)
.\scripts\build.ps1                  # add -Config Debug / -Clean / -NoAirPlay / -NoMiracast
.\build\Release\1PhoneMirror.exe
```

The exe name on disk is `1PhoneMirror.exe`; older docs/scripts sometimes spell
it lowercase — Windows treats them as equal but match the CMake target
(`1PhoneMirror`) in new code.

There is **no automated test suite**. Closest thing to a unit test is
`1PhoneMirror.exe --srp-self-test` (returns 0 on success). Always smoke-test
a Release build before tagging.

## Repository layout

| Path | Contents |
|---|---|
| [include/opm/](include/opm) | Public headers — namespace `opm::`, one subdir per module (`airplay/`, `android/`, `cast/`, `media/`, `miracast/`, `network/`). |
| [src/](src) | Implementations mirror the header layout. `main.cpp` → CLI/startup; `app.cpp` → `opm::App` glues renderer + receivers. |
| [lib/](lib) | Vendored sources: `stb_image*` and `playfair/` (AirPlay FairPlay). Do not reformat. |
| [resources/](resources) | `.rc`, app manifest, icons embedded into the exe. |
| [installer/](installer) | WiX 5 `.wxs` source + `THIRD_PARTY_LICENSES.txt`. |
| [scripts/](scripts), [package.ps1](package.ps1) | Build + MSI/Intune packaging. |
| [telemetry/](telemetry) | **Separate** .NET 8 Azure Functions project — deployed via `azd up`, not part of the C++ build. See [telemetry/README.md](telemetry/README.md). |
| [manifests/](manifests) | winget manifest fork copy (publish via [.github/workflows/winget.yml](.github/workflows/winget.yml)). |
| [docs/screenshots/](docs/screenshots) | README screenshots only. |

## Architecture rules

- **One protocol streams at a time.** `opm::App::active_source_` is an atomic
  enum (`None | AirPlay | Miracast | Cast | Android`). Receiver callbacks must
  check `active_source_` before pushing frames to the renderer/audio.
- **Per-protocol gating.** Every protocol's headers, sources, link libs, and
  call sites are wrapped in `#ifdef ENABLE_AIRPLAY` / `_MIRACAST` / `_CAST` /
  `_ANDROID`. The matching `option()` in [CMakeLists.txt](CMakeLists.txt)
  controls both the source list and the compile definition — keep them in
  sync when adding files.
- **No blocking on the SDL event/render thread.** Disconnects, `adb` shells,
  and anything that joins worker threads must run on a detached `std::thread`
  (see the `airplay_.disconnect_source` / `android_disconnect` call sites in
  `src/app.cpp`). Blocking the UI thread surfaces as "Not Responding" and
  leaves iOS unable to reconnect.
- **Multi-device support.** AirPlay can hold multiple sources; Android keeps
  one `AndroidSession` per serial in `scrcpy_sessions_`. The bottom-bezel dot
  picker is driven by `App::source_order_` — preserve insertion order.

## Conventions

- **Version is set in ONE place:** the `project(... VERSION x.y.z ...)` line
  in [CMakeLists.txt](CMakeLists.txt). It feeds `OPM_VERSION_{MAJOR,MINOR,PATCH}`
  compile defs and is parsed by [package.ps1](package.ps1) for the MSI name.
  When bumping it, **also add a row to the version-history table in
  [README.md](README.md)** — the in-app "Version history" panel (`V`) is
  rendered from that same table.
- **Logging:** plain `std::cout` / `std::cerr` with a `[Module] ` prefix
  (`[App]`, `[AirPlay]`, `[Shutdown]`, `[CRASH]`, …). `opm::LogBuffer` tees
  stdout/stderr into the in-app log viewer (`L` key); do not introduce a
  separate logging framework.
- **Settings persistence:** add fields to `opm::Settings` in
  [include/opm/settings.h](include/opm/settings.h), then extend the parser
  and writer in [src/settings.cpp](src/settings.cpp). File format is plain
  `key=value` lines in `%APPDATA%\1PhoneMirror\settings.ini`. Always provide
  a sensible default so existing users don't lose their config.
- **Strings stay ASCII** in code and CLI output (the log viewer renders with
  SDL_ttf; non-ASCII glyphs show as boxes).
- **Includes:** use `<opm/...>` for project headers (configured as
  `target_include_directories(... PRIVATE include)`).

## Known pitfalls (already coded around — don't undo)

- **Stale process / port conflict.** `kill_stale_instances()` in
  [src/main.cpp](src/main.cpp) terminates leftover `1PhoneMirror.exe`
  processes before binding ports 7000/7100 and registering mDNS. Required
  after a crash.
- **`adb.exe` squats on UDP 5353.** `kill_adb_processes()` in
  [src/app.cpp](src/app.cpp) force-kills all `adb.exe` instances on shutdown
  so Bonjour can rebind mDNS on the next launch.
- **Firewall prompt.** `check_firewall_rules()` creates inbound TCP 7000/7100,
  UDP 5353/7010/7011 and a program-level rule via an elevated `netsh`. Must
  remain idempotent.
- **TripIt 1920×1080 AirPlay surface.** The renderer letterboxes it inside

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MSEndpointMgr/1PhoneMirror](https://github.com/MSEndpointMgr/1PhoneMirror) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
