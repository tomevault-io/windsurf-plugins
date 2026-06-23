---
trigger: always_on
description: This file gives Claude Code practical guidance for working in this repository. It is intended for maintainers and coding agents, not as end-user documentation.
---

# CLAUDE.md

This file gives Claude Code practical guidance for working in this repository. It is intended for maintainers and coding agents, not as end-user documentation.

## Project Overview

This project is a Python-based NetEase Cloud Music MCP controller. It exposes music automation features to MCP clients through FastMCP.

The project combines three main control paths:

- NetEase Cloud Music URL schemes such as `orpheus://` for launching the desktop client and playing songs or playlists.
- Global hotkeys through `pyautogui` for playback, track navigation, volume, lyrics, liking songs, and mini mode.
- Selenium automation against the NetEase Cloud Music desktop client running with a remote debugging port, used for daily recommendations and private roaming.

Windows is the primary target platform. macOS supports the basic hotkey and URL-scheme features, while daily recommendations and private roaming are Windows-oriented and may not work on macOS.

## Technology Stack

- Python: `>=3.10`
- Package manager: `uv`
- MCP framework: `fastmcp>=2.0.0`
- Desktop automation: `pyautogui`
- Windows integration: `pywin32`
- Process management: `psutil`
- Browser automation: `selenium`
- HTTP requests: `requests`

Dependency declarations live in [pyproject.toml](pyproject.toml). The resolved dependency lockfile is [uv.lock](uv.lock).

## Common Commands

### Set Up The Environment

```powershell
uv sync
```

Install development dependencies when needed:

```powershell
uv sync --extra dev
```

### Run The MCP Server

```powershell
uv run src/server.py
```

Or run the installed project script:

```powershell
uv run cloudmusic-auto-player
```

The script registered in [pyproject.toml](pyproject.toml) is `cloudmusic-auto-player`. Do not use the older name `cloudmusic-mcp`.

### Code Quality

```powershell
uv run black src/
uv run isort src/
uv run flake8 src/
uv run mypy src/
```

### Tests

```powershell
uv run pytest
```

At the time of writing, this repository may not contain a `tests/` directory. If no tests exist, `pytest` can fail because the configured test path is missing or because there are no collected tests. Add focused tests when changing core logic.

### Dependency Maintenance

```powershell
uv add <package-name>
uv add --dev <package-name>
uv lock --upgrade
```

## Repository Layout

```text
CloudMusic_Auto_Player/
├── src/
│   ├── server.py                     # FastMCP server entry point and MCP tool definitions
│   ├── controllers/
│   │   ├── netease_controller.py      # URL schemes, hotkeys, and window minimization
│   │   └── daily_controller.py        # Selenium automation for daily recommendations and roaming
│   ├── utils/
│   │   ├── config_manager.py          # Configuration loading, saving, and platform detection
│   │   └── music_search.py            # NetEase search API integration and play URL generation
│   ├── config/
│   │   ├── hotkeys.json               # Windows and macOS hotkey configuration
│   │   └── README.md                  # Hotkey configuration notes
│   └── chromedriver/
│       └── win64/chromedriver.exe     # Bundled Windows ChromeDriver
├── config.json                        # MCP client example or compatibility config
├── netease_config.json                # NetEase client path, debug port, and ChromeDriver path
├── playlists.json                     # System and user playlist configuration
├── pyproject.toml                     # Project metadata, dependencies, and tool config
├── PROJECT_STRUCTURE.md               # Architecture and structure notes
└── README.md                          # End-user documentation
```

## Core Modules

### [src/server.py](src/server.py)

`server.py` is the MCP-facing boundary. It is responsible for:

- Creating the `FastMCP("网易云音乐控制器")` server instance.
- Initializing `NeteaseMusicController`.
- Registering all MCP tools.
- Returning consistent dictionaries such as `{"success": ..., "data": ..., "message": ...}`.

Development guidance:

- Put business logic in `controllers/` or `utils/` first, then expose it through a thin MCP tool in `server.py`.
- Avoid adding complex Selenium, search, or configuration logic directly to `server.py`.
- Keep MCP tool responses structured. Failure paths should include a clear `error` or `message`.

### [src/controllers/netease_controller.py](src/controllers/netease_controller.py)

This module handles basic desktop-client control:

- `launch_by_url_scheme()` tries `os.startfile`, `subprocess`, and `webbrowser` to open NetEase URL schemes.
- `send_global_hotkey()` sends configured hotkeys through `pyautogui.hotkey()`.
- `_minimize_netease_window()` uses `win32gui` on Windows to find and minimize NetEase Cloud Music windows.

Important notes:

- This code affects the real desktop session. Be careful when testing changes that send hotkeys.
- `pyautogui.FAILSAFE = False` is set, so shortcut changes should be reviewed carefully.
- `win32gui` is Windows-only. Do not assume window control is available on all platforms.

### [src/controllers/daily_controller.py](src/controllers/daily_controller.py)

This module handles advanced Selenium-based features:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SpongeBaby-124/CloudMusic_Auto_Player](https://github.com/SpongeBaby-124/CloudMusic_Auto_Player) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-23 -->
