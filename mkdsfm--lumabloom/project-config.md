---
trigger: always_on
description: This file gives Codex and similar coding agents repo-specific guidance for working safely and efficiently in this project.
---

# AGENTS.md

This file gives Codex and similar coding agents repo-specific guidance for working safely and efficiently in this project.

## Purpose

`brightness-sensor` is a mixed hardware/software repository with one ESP32-C6 firmware track and one Windows desktop application:

- `firmware/firmware_esp32c6/` - `ESP-IDF` firmware for `Waveshare ESP32-C6-LCD-1.47` with `KY-018`
- `hardware/` - wiring, BOM, assembly checks, revision notes, and printable enclosure assets
- `pc-app/` - Windows-only `.NET` console application that reads telemetry from a serial port and controls monitor brightness through WMI
- `docs/` - getting started, firmware, build, protocol, profiles, and workflow notes

The current documented hardware and enclosure flow targets `ESP32-C6`.

## Read This First

Before making non-trivial changes, read the smallest relevant set of files:

1. Root [`README.md`](README.md)
2. The target-specific docs:
   - [`docs/getting-started.md`](docs/getting-started.md)
   - [`docs/firmware.md`](docs/firmware.md) when touching firmware
   - [`docs/build.md`](docs/build.md) when touching the Windows app build/run flow
   - [`docs/protocol.md`](docs/protocol.md)
   - [`docs/device-profiles.md`](docs/device-profiles.md) when profile logic is involved
   - [`hardware/README.md`](hardware/README.md) and [`hardware/WIRING.md`](hardware/WIRING.md) when hardware wiring or assembly is involved
3. Contribution workflow when changing project structure, validation rules, or contributor-facing docs:
   - [`CONTRIBUTING.md`](CONTRIBUTING.md)
4. Firmware-specific README when touching firmware:
   - [`firmware/firmware_esp32c6/README.md`](firmware/firmware_esp32c6/README.md)
5. Solution and test projects when touching `pc-app/`:
   - [`pc-app/brightness-sensor.sln`](pc-app/brightness-sensor.sln)

If the user mentions Codex skills or release workflows, also read [`docs/skills-for-users.md`](docs/skills-for-users.md).

## Project Truths

- `pc-app` is Windows-only by design.
- The serial protocol is newline-delimited JSON; see `docs/protocol.md`.
- `ESP32-C6` firmware sends calibrated normalized readings in `value` in the `0..1000` range and may also send `raw`.
- `pc-app` resolves built-in runtime defaults from `deviceId + sensorId`, then applies user overrides from `appsettings.json`.
- For `ESP32-C6`, startup calibration is part of the expected runtime contract. `pc-app` sends a `calibrate` command and the device stays effectively `UNCAL` until calibration completes.

## Repo Map

### `pc-app/`

Main solution:

- `BrightnessSensor.ConsoleApp/` - entry point, orchestration, config loading, profile resolution
- `BrightnessSensor.BrightnessMath/` - mapping and smoothing logic
- `BrightnessSensor.DeviceReading/` - telemetry parsing and serial-related code
- `BrightnessSensor.WindowsBrightness/` - Windows brightness control integration
- `BrightnessSensor.ConsoleApp.Tests/` - config/profile/processing tests
- `BrightnessSensor.DeviceReading.Tests/` - parser and discovery tests

### `firmware/firmware_esp32c6/`

- `ESP-IDF` project
- Uses onboard LCD plus `KY-018`
- Main configurable constants live in `main/app_config.h`
- UI placeholders in `main/ui_generated_screen.c` are part of the dynamic display contract

## Safe Working Rules

- Do not change `deviceId`, `sensorId`, measurement kind, or calibration flow casually; these affect cross-component compatibility.
- Do not edit generated/build artifacts unless the user explicitly asks for that.
- Prefer editing source files rather than anything under `build/`, `bin/`, `obj/`, or generated outputs.
- Treat `pc-app/appsettings.json` as a local machine file. It is intentionally ignored by git and should usually not be created or modified unless the user explicitly wants a local run configuration.
- `sdkconfig` and `sdkconfig.old` are ignored generated/config files for `ESP-IDF`; avoid changing them unless the task is specifically about project configuration.

## Files Usually Safe To Edit

- Documentation in `docs/`
- Hardware documentation in `hardware/`
- `.cs` files and `.csproj` files in `pc-app/`
- Tests in `pc-app/*Tests/`
- `firmware/firmware_esp32c6/main/*.c` and `main/*.h`
- `firmware/firmware_esp32c6/CMakeLists.txt` and related source-level build files

## Files To Avoid Editing Without Strong Reason

- `pc-app/appsettings.json`
- `**/build/**`
- `**/bin/**`
- `**/obj/**`
- `firmware/firmware_esp32c6/sdkconfig`
- `firmware/firmware_esp32c6/sdkconfig.old`
- `firmware/firmware_esp32c6/managed_components/**`

## Common Tasks

### When changing `pc-app`

Typical commands from `pc-app/`:

```powershell
dotnet restore
dotnet build brightness-sensor.sln
dotnet test brightness-sensor.sln
dotnet run --project BrightnessSensor.ConsoleApp
```

Focus areas:

- profile resolution
- config validation
- smoothing / brightness mapping behavior
- serial discovery and telemetry parsing

When changing profile logic, also verify that docs and examples still match the effective behavior.

### When changing `ESP32-C6` firmware

Typical commands from `firmware/firmware_esp32c6/` in an `ESP-IDF` shell:

```powershell
idf.py set-target esp32c6
idf.py build
idf.py -p COMx flash monitor
```

For merged release binaries:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mkdsfm/LumaBloom](https://github.com/mkdsfm/LumaBloom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
