---
trigger: always_on
description: > **Important:** After modifying any project content (code, config, structure, dependencies), you **MUST** update this file to reflect the changes. Keep this document perfectly synchronized with the actual project state at all times.
---

# AGENTS.md

> **Important:** After modifying any project content (code, config, structure, dependencies), you **MUST** update this file to reflect the changes. Keep this document perfectly synchronized with the actual project state at all times.

---

## Project Overview

**ahko** is a keyboard-driven quick launcher for Windows, built with **AutoHotkey v2.0**. It watches a user-configured folder and presents a grid overlay UI to launch files, folders, and shortcuts via keyboard shortcuts. Invoked by a configurable global hotkey (default `Alt+Q`).

- **Author:** Nigh (HelloWorks)
- **License:** MIT
- **Current Version:** `1.0.3` (defined in `meta.ahk`)
- **Repository:** https://github.com/Nigh/ahko

---

## Directory Structure

```
ahko/
├── app.ahk                      # Entry point - main application bootstrap
├── meta.ahk                     # App metadata (name, version, filenames, URLs, changelog)
├── ahko.ahk                     # Core logic: folder scanning, key binding, item initialization
├── ahko_ui.ahk                  # UI dispatcher (GDIp grid view)
├── ahko_gridview_ui.ahk         # GDIp-based grid view UI class (main launcher UI)
├── setup/
│   ├── ahko_setup_gui.ahk       # Setup window logic (ahk-xaml / WPF)
│   └── setup.axml               # Setup UI layout (AXML markup)
├── lib/
│   └── ahk-xaml/                # Vendored ahk-xaml framework (https://github.com/owhs/ahk-xaml)
│       ├── XAML_Host.ahk        # WPF IPC bridge and engine host
│       ├── XAML_Generator.ahk   # Chainable XAML AST builder
│       ├── XAML_GUI.ahk         # High-level window scaffolding
│       ├── XAML_Components.ahk  # UI components (HotKeyBox, etc.)
│       ├── AXML.ahk             # AXML parser and state binding
│       ├── dep/                 # C# WPF bridge source and component BAML
│       └── shaders/             # Optional WPF shader assets
├── themes.ini                   # ahk-xaml theme tokens (from upstream examples)
├── Gdip_All.ahk                 # GDI+ library for AHK v2 (3rd party, ~3030 lines)
├── isFullScreen.ahk             # Fullscreen window detection utility class
├── tray.ahk                     # System tray menu setup
├── update.ahk                   # Auto-update logic (checks GitHub releases)
├── updater.c                    # C source for updater binary (extracts zip, launches exe)
├── updater.h                    # C header for updater (filename defines)
├── distribution.ahk             # Build/distribution script (compile + package)
├── set_auto_run.ahk             # Windows startup helper (Task Scheduler, admin, no UAC)
├── icon.ico                     # Application icon (ICO)
├── icon.png                     # Application icon (PNG)
├── LICENSE                      # MIT license
├── README.md                    # Project documentation
├── AGENTS.md                    # This file - agent instructions
├── .gitignore                   # Ignores: dist/, *.exe, *.ini, compile_prop.ahk, .vscode/
├── .gitmodules                  # Git submodule: ahk-compile-toolset
├── .github/
│   └── workflows/
│       └── release.yml          # CI: auto-release on v* tag push
├── assets/                      # Screenshots for README
│   ├── setup.png
│   └── setupui.png
└── ahk-compile-toolset/         # Git submodule - compile toolchain
    ├── Ahk2Exe.exe              # AHK script-to-exe compiler
    ├── AutoHotkey64.exe         # AHK v2 runtime for compilation
    ├── mpress.exe               # Executable compressor
    ├── tcc/                     # Tiny C Compiler (for building updater.c)
    └── c_utils/                 # C utility files
```

---

## Architecture

### Execution Flow

```
app.ahk (entry point)
  ├── meta.ahk              (metadata: version, filenames, URLs)
  ├── update.ahk            (auto-update check against GitHub releases)
  │   └── Uses MSXML2.ServerXMLHTTP to fetch version.txt
  │   └── Downloads new zip, launches updater.c binary to extract
  ├── ahko.ahk              (core logic)
  │   ├── isFullScreen.ahk  (multi-monitor fullscreen detection)
  │   ├── setup/ahko_setup_gui.ahk (settings GUI via ahk-xaml)
  │   │   ├── lib/ahk-xaml/ (vendored WPF framework)
  │   │   └── setup/setup.axml (AXML layout)
  │   └── ahko_ui.ahk       (UI dispatcher)
  │       └── ahko_gridview_ui.ahk (GDIp grid view class)
  │           └── Gdip_All.ahk (GDI+ library)
  └── tray.ahk              (system tray menu)
```

### Module Responsibilities

| Module | Responsibility |
|--------|----------------|
| `app.ahk` | Entry point. Bootstraps app, includes all modules, sets up dev hotkeys (F5=exit, F6=reload) |
| `meta.ahk` | Single source of truth for app name, version, binary filename, download URL, changelog |
| `ahko.ahk` | Core logic. Reads `setting.ini`, scans watch folder (2 levels, 16 items max per level), builds item array. Detects UAC mode (`A_IsAdmin`), provides `ahko_invoke` wrapper that falls back to setup GUI when grid is empty in UAC mode |
| `ahko_ui.ahk` | Thin dispatcher: initializes grid view class with GDIp rendering |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nigh/ahko](https://github.com/Nigh/ahko) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
