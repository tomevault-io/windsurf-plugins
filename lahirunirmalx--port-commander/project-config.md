---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & run

```bash
cmake -S . -B build
cmake --build build
./build/apcommander                    # dashboard — click a tile to launch a tool
./build/portcommander                  # standalone: network-port inspector
./build/wificommander                  # standalone: nmcli Wi-Fi / hotspot GUI
./build/adc_gui                        # standalone: 24-bit ADC monitor
./build/psu_gui                        # standalone: dual-channel PSU full GUI
./build/psu_gui_single                 # standalone: single-channel PSU full GUI
./build/psu_gui_toolbar                # standalone: dual-channel PSU compact strip
./build/psu_gui_toolbar_single         # standalone: single-channel PSU compact strip
```

System dependencies (Debian/Ubuntu): `build-essential cmake pkg-config libsdl2-dev libsdl2-ttf-dev lsof procps fonts-dejavu-core`. `wificommander` additionally requires `network-manager` (`nmcli`) at runtime, and `qrencode` to display the join-QR (silently degrades when missing). The ADC/PSU tools talk to a serial device (default `/dev/ttyUSB0`, override via argv[1]) and fall back to a demo mode when the port can't be opened.

There is no test suite, no linter config, and no formatter config in the repo. The compile flags `-Wall -Wextra -Wpedantic` are the only enforced checks for the native targets (`portcommander`, `wificommander`, `apcommander`). The imported `adc_gui` / `psu_gui*` targets were brought in with their upstream warnings still tripping (e.g. `usleep` deprecation under `_POSIX_C_SOURCE=200809L`); CMakeLists.txt suppresses a handful of `-Wno-unused-*` and `-Wno-sign-compare` for those targets only — don't propagate those suppressions to the native code. CI just runs `cmake --build` (`.github/workflows/build.yml`) and uploads all eight binaries as one artifact.

## Architecture

Eight executables and one static library, built from one CMakeLists.txt:

- **`apcompat`** (static lib in `compat/`) is the cross-platform abstraction layer used by the dashboard. It exposes: `compat_app_dir` (wraps `SDL_GetBasePath`), `compat_exe_suffix`, `compat_can_execute`, `compat_path_lookup` (walks `$PATH` for a bare name), `compat_home_dir` (HOME with USERPROFILE fallback on Windows), `compat_default_font` (platform-specific monospace-font search), and the spawn/poll pair `compat_spawn` (accepts an optional argv array) / `compat_proc_poll` / `compat_proc_free`. POSIX path uses `fork` + `execvp` + non-blocking `waitpid`; the Windows path (sketched, uncompiled) uses `CreateProcessA` + `WaitForSingleObject` + `GetExitCodeProcess` and naïvely quotes argv into a cmdline. The POSIX child also closes inherited FDs above stderr before `execvp` so spawned tools don't keep the dashboard's X11 socket / font mmaps alive. **All non-portable code in the dashboard goes through this layer** — adding Windows means filling in the `_WIN32` branches there, not touching the dashboard's source files.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lahirunirmalx/port-commander](https://github.com/lahirunirmalx/port-commander) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
