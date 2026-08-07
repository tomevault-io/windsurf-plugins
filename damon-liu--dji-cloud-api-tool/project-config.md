---
trigger: always_on
description: > Keep `CLAUDE.md` in sync with this file — it's a copy used by Claude Code.
---

# AGENTS.md

> Keep `CLAUDE.md` in sync with this file — it's a copy used by Claude Code.

## Build

```bash
# Windows (MinGW) — Debug
cmake -B build_mingw -G "MinGW Makefiles" -DCMAKE_BUILD_TYPE=Debug
cmake --build build_mingw

# Windows — deploy Qt DLLs for distribution
cmake --build build_mingw --target deploy

# One-click release packaging (build + deploy + strip credentials + zip)
# Requires bash: Git Bash, WSL, or MSYS2
bash package.sh v1.0.3

# Linux (native)
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build

# Windows → Linux cross-compile (Zig) — NOTE: cmake/toolchains/linux-x64.cmake
# is NOT in the repo; you must supply it externally.
export ZIG_PATH=/path/to/zig/zig.exe
cmake -B build_linux -G "MinGW Makefiles" \
      -DCMAKE_TOOLCHAIN_FILE=cmake/toolchains/linux-x64.cmake \
      -DZIG_PATH="$ZIG_PATH" -DCMAKE_BUILD_TYPE=Release
cmake --build build_linux
```

- C++17, Qt 6 (Core, Widgets, Mqtt), CMake ≥ 3.10
- Binary: `DjiCloudApi.exe` (Windows), `main` (Linux cross-compile)
- `cmake/toolchains/linux-x64.cmake` is **not committed** — the Zig cross-compile path needs it externally
- CMake copies `config/topic_mappings.json` → build dir on **every** build
- CMake copies `src/resources/config.json` → build dir `config.json` **only on first build** (preserves credentials)
- `package.sh` replaces `config.json` with `deploy/config.example.json` to strip credentials before zipping
- Build output goes to `build_mingw/`; `deploy/` holds pre-built distribution artifacts (exe is committed, DLLs are gitignored)
- No lint, typecheck, or test commands exist in this project

## Config file resolution

At runtime (see `src/main.cpp`):
1. `<exe_dir>/config/config.json` (primary — `config/` subdir is auto-created on startup)
2. `./config.json` (CWD-relative fallback for backward compat)

The CMake template `src/resources/config.json` is copied to the build root. At runtime the app may move/create it inside `config/`. When editing config, check both locations.

## Architecture

Three-layer, single-threaded — Qt signals/slots for async I/O:

```
UI Layer      src/ui/     — MainWindow, DeviceTreeWidget, OsdPanel, TopicParsePanel,
                            RawJsonPanel, PublishPanel, ConfigDialog, TopicListWidget,
                            DockControlPanel, FlightControlPanel, MaintenancePanel
Core Layer    src/core/   — DeviceManager (central dispatcher), ConfigStore, TopicManager,
                            TopicMapping, DeviceInfo/OsdData structs,
                            DockCommand/DockCommandExecutor (command system)
MQTT Layer    src/mqtt/   — MqttClientManager (QMqttClient wrapper, exponential backoff reconnect)
```

**Data flow:** MQTT message → `MqttClientManager::messageReceived` → `DeviceManager::parseAndRoute()` → `TopicManager` matches topic to device SN → parses DJI-format JSON `{"tid":..., "data":...}` → caches OSD, emits `deviceOsdUpdated(sn, rawJson)` → UI panels reactively update.

**Thread model:** Everything runs on the Qt main thread. `QMqttClient` is async — no worker threads.

**UI language:** All labels, field names, enum values are Chinese (UTF-8). Compiler flags enforce UTF-8 encoding: `/utf-8` (MSVC), `-fexec-charset=UTF-8` (GCC/MinGW).

### Key classes

| Class | Role |
|---|---|
| `DeviceManager` | Central dispatcher. Owns ConfigStore, TopicManager, MqttClientManager. Device CRUD, message routing, OSD caching (field-level merge via `mMergedOsdData`), profile switching. |
| `ConfigStore` | JSON config persistence with multi-profile support. Each profile has independent MQTT config + devices + topics. **Auto-saves on every mutation** — do not batch writes without considering this. |
| `TopicManager` | Topic ↔ device SN bidirectional mapping. Manages enabled/disabled state per topic, emits `topicsChanged` to trigger MQTT re-subscribe. |
| `MqttClientManager` | `QMqttClient` wrapper. Exponential backoff reconnect (base 1s, max 30s). Dedup subscription management. Tracks pending publishes via messageId→topic map. |
| `TopicMapping` | Loads `config/topic_mappings.json` — maps MQTT topic patterns to Chinese field names, units, enum translations, and group layouts. Topic keys prefixed with `dock/` or `aircraft/` for device-type-specific mappings. |
| `DockCommandExecutor` | Serial command executor. Publishes to `services` topic → subscribes `services_reply` → matches by `tid` → 10s timeout. Only one pending command at a time. |
| `MainWindow` | Top-level window (1280×760). Toolbar with config/connect/disconnect. Horizontal splitter: left = device tree + topic list (vertical split), right = OSD + TopicParse + RawJson + Publish (tabbed/stacked). Dock/Flight/Maintenance control panels open as separate dialogs from the "功能中心" toolbar menu. |

### Device hierarchy

`DeviceInfo` has `parentSn` — dock devices are top-level, aircraft can be children of a dock (`isChild()`). Tree renders as two-level hierarchy. Child aircraft are auto-discovered from dock OSD messages (`DeviceManager::checkAndAddChildAircraft`).

### OSD field-level merge

DJI sends OSD fields split across multiple MQTT messages. `DeviceManager` uses `mMergedOsdData` to merge fields incrementally — the latest value for each field key wins. The UI sees the merged result.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [damon-liu/Dji-cloud-api-tool](https://github.com/damon-liu/Dji-cloud-api-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
