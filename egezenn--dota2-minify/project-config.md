---
trigger: always_on
description: `dota2-minify` is a modding tool and manager for Dota 2, designed to streamline the modification of game files, UI configurations, styling, and application of specific game patches.
---

# Dota2 Minify

`dota2-minify` is a modding tool and manager for Dota 2, designed to streamline the modification of game files, UI configurations, styling, and application of specific game patches.

> [!NOTE]
> This file serves as a comprehensive technical guide for both **AI Agents** and **Human Contributors**. It outlines the project's soul, its structural DNA, and the rules that govern its evolution. Whether you are a machine processing these tokens or a human reading these lines, welcome to the team!

## Tech Stack

- **Language**: Python 3.13
- **Package Manager**: `uv`
- **GUI Framework**: DearPyGui (DPG)

## Core Directory Structure

- `Minify/`: The main application package.
  - `__main__.py`: The entry point for the DearPyGui application.
  - `cli.py`: The entry point for the Headless CLI interface.
  - `patch/`: Package handling the compilation, extraction, and patching of Mod files.
    - `__init__.py`: Core patching orchestration logic.
    - `blacklist.py`: Logic for processing mod blacklist rules.
    - `styling.py`: Logic for applying mod CSS styles.
    - `replacer.py`: Logic for file replacement rules via CSV.
    - `vpk_utils.py`: Utility functions for VPK operations and extraction.
    - `manifest_utils.py`: Utilities for loading mod manifests and versioning.
    - `unins.py`: Logic for mod uninstallation and cleanup.
  - `conditions.py`: System conditionals (verifying paths, workshop tools status).
  - `helper.py`: Provides utility functions to compile images, execute mod scripts dynamically, etc.

  - `bin/`: Contains static utility files such as `localization.json` and `gamepakcontents.txt` (a complete list of files inside the game pak, created once a patch is ran).

  - `browsers/`: Package for third-party mod browsers (e.g., D2PFX).
    - `d2pfx/`: Specific implementation for the D2PFX mod browser.

  - `core/`: Contains fundamental backend modules:
    - `base.py`: Unchanging static variables, paths, and OS-level info.
    - `config.py`: Utilities for reading and writing JSON config files (`minify_config.json`, `mods.json`).
    - `constants.py`: Important static paths and pre-calculated lists for the patching pipeline.
    - `fs.py`: File system utilities, path manipulation, reading/writing/copying files.
    - `log.py`: Handles unhandled exceptions and writes warnings/crashes to log files.
    - `mods_shared.py`: Shared capabilities specific to handling mods logic.
    - `registry.py`: Central registry for browsers and plugins.
    - `steam.py`: Functions to detect Steam directories, game paths, and modify launch options.
    - `utils.py`: Shared utilities
    - `vpk_utils.py`: Utility functions for VPK operations and metadata generation.
    - `output.py`: Agnostic communication interface between backend and UI/CLI.

  - `ui/`: Contains the DearPyGui interface logic:
    - `announcements.py`: Fetches and displays global announcements to users on app start.
    - `checkboxes.py`: Logic for rendering and managing the state of mod enablement checkboxes.
    - `details.py`: Renders the detailed view of an individual mod (parsing `notes.md` and preview image).
    - `dev_tools.py`: Helper GUI functionalities intended for developers or advanced debugging.
    - `fonts.py`: Registers and initializes different custom fonts for DearPyGui.
    - `gui.py`: Manages the overall layout logic, viewport scaling, and rendering routines.
    - `localization.py`: Multi-language dynamic text localization support.
    - `markdown.py`: Custom parser to render markdown files (`notes.md`) using DPG items.
    - `modal_shared.py`: Base components for pop-up dialogs and modals.
    - `modals.py`: Implementations of specific modals (Uninstall, Announcements, Update dialogs).
    - `settings.py`: The powerhouse for rendering the global settings menu, including the dynamic generation of mod-specific configuration options.
    - `shared.py`: Stores minimal state shared across UI modules.
    - `terminal.py`: Draws the "terminal" window in the UI that logs the patching progress in real-time.
    - `theme.py`: Configures the DearPyGui color maps, styles, and dark-theme configurations.
    - `window.py`: Window focus logic and drag/drop/resize helper implementations.
  - `mods/`: The root directory for all available mods native to Minify. Each subdirectory represents a standalone mod.

### Third-Party Dependencies

The application uses external executables downloaded at runtime into the `Minify/` directory:

- **Ripgrep (`rg.exe`)**: Utilized for extremely fast, pattern-based text searching and filtering during the compilation and patching processes. May use it from system if existent.
- **Source2Viewer (`Source2Viewer-CLI.exe`)**: Used to parse, decompile, or convert proprietary Source 2 engine assets into usable formats. Downloaded only if Workshop Tools are available.

Additionally, it interacts with Dota 2 Workshop Tools if the DLC is installed:

- **ResourceCompiler (`resourcecompiler.exe`)**: Invoked by Minify to compile raw assets (XML, CSS etc.) into their Source 2 binary equivalents (`_c`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Egezenn/dota2-minify](https://github.com/Egezenn/dota2-minify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
