---
trigger: always_on
description: This document describes the implemented native architecture, including Step 1 (storage isolation) and Step 2 (background lifetime). Source code and tests remain authoritative. Planned packaging behavior is distinguished from implemented behavior.
---

# Biomes Engine — Complete Technical Architecture & Developer Specification

This document describes the implemented native architecture, including Step 1 (storage isolation) and Step 2 (background lifetime). Source code and tests remain authoritative. Planned packaging behavior is distinguished from implemented behavior.

---

## 1. High-Level Architecture & Tech Stack

Biomes is a lightweight, high-performance Windows desktop application manager built on a **hybrid Win32 + WebView2 architecture**.

~~~text
StartupOptions -> SingleInstance -> AppPaths -> Migration -> NativeSettings
                                      |
                       BackgroundHost (hidden top-level HWND)
                       process loop / global hotkeys / tray
                                      |
                  main.cpp orchestration and trusted-page JSON IPC
                      |                              |
                WebViewWindow                  Native engine
                (dashboard)       WindowScaler / AppLauncher / MonitorManager
                                  JsonManager / GridOverlay / LaunchPanel
~~~

* **Language Standard:** Modern C++17 compiled via MSVC (Visual Studio 2022/2026).
* **Native Win32 Libraries:** `user32`, `gdi32`, `shell32`, `advapi32`, `ole32`, `dwmapi`, `shcore`, `dcomp`.
* **Third-Party Libraries (Zero heavy frameworks):**
  * `nlohmann/json` (header-only JSON serialization).
  * `Microsoft.Web.WebView2` (Native COM interface bindings without WRL dependencies).
* **UI Hosting:** WebView2 renders `frontend/index.html` and separate scripts/styles, copied beside the executable by `BiomesAssets`. The dashboard does not own process lifetime; `BackgroundHost::Run()` does.

---

## 2. Directory, Storage & Path Isolation

### A. Project modules

~~~text
biomes/
|-- CMakeLists.txt                 # Native sources, libraries, asset copying, tests
|-- AGENTS.md
|-- WebView2Loader.dll
|-- resources/                    # Native icon resources
|-- frontend/                     # Frozen HTML/CSS/JS and static assets
|   |-- index.html, app.js, save-dialog.js, newsletter.js, styles.css
|   |-- launch-panel.html, launch-panel.css, launch-panel.js
|   |-- onboarding.css, onboarding.js
|   `-- assets/, logo/, images/, cardsimages/, icons/
|-- include/
|   |-- core/
|   |   |-- app_launcher.hpp          # Executable, packaged-app and URI launches
|   |   |-- app_paths.hpp             # Absolute per-user storage paths
|   |   |-- background_host.hpp       # Hidden lifetime/hotkey owner
|   |   |-- hotkey_manager.hpp        # Shortcut parsing and registration mapping
|   |   |-- json_manager.hpp          # Biome persistence and topology variants
|   |   |-- launch_progress.hpp       # Launch progress state
|   |   |-- legacy_data_migration.hpp # Non-destructive copy-and-switch migration
|   |   |-- monitor_manager.hpp       # Display identity and work areas
|   |   |-- native_settings.hpp       # Validated backend settings
|   |   |-- single_instance.hpp       # Mutex and activation handoff
|   |   |-- startup_options.hpp       # Silent startup flags
|   |   |-- startup_registration.hpp  # Current-user Run registry entry
|   |   `-- window_scaler.hpp         # Discovery, tracking, placement, restoration
|   |-- ui/
|   |   |-- grid_overlay.hpp          # Zone creation and app binding
|   |   |-- launch_panel.hpp          # Launch-progress WebView2 host
|   |   |-- tray_manager.hpp          # Notification icon and context menu
|   |   `-- webview_window.hpp        # Dashboard HWND and IPC
|   `-- external/                    # JSON/WebView2 dependencies
|-- src/
|   |-- main.cpp
|   |-- core/                        # Corresponding compiled core implementations
|   `-- ui/                          # Overlay, launch panel, tray, dashboard
`-- tests/
    |-- stability_tests.cpp
    |-- storage_tests.cpp
    |-- lifecycle_tests.cpp
    `-- launch_panel_preview.cpp
~~~

There is no separate `hotkey_host.hpp`: `BackgroundHost` owns that responsibility. CMake includes all compiled Step 1/2 source modules.

### B. Installation versus user data

The intended installer location is `%LOCALAPPDATA%\Programs\biomes\`. This is a planned packaging destination, not an installer already implemented by Steps 1/2. Development binaries run from CMake build output; asset lookup uses the actual executable directory.

~~~text
%LOCALAPPDATA%\Programs\biomes\    # Planned replaceable installation files
    Biomes.exe
    WebView2Loader.dll
    index.html, scripts, styles, fonts, bundled images, ...

%LOCALAPPDATA%\biomes\             # Implemented stable user-data location
    config\
        biomes.json
        settings.json
        settings.lock
        settings.json.tmp          # Transient settings-save staging file
    logs\
        biomes_runtime.log
        legacy-biomes_runtime.log  # If imported
    webview_data\                  # Both WebView2 hosts, cache and local storage
    images\                        # User-owned image storage directory
    backups\
        migration.lock
        legacy-migration-v1.done

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AbdelGhafourRebbouh/biomes](https://github.com/AbdelGhafourRebbouh/biomes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
