---
trigger: always_on
description: **KNX-Lens** is a suite of Python tools for monitoring and analyzing KNX bus traffic. It consists of three main components:
---

# KNX-Lens Copilot Instructions

## Project Overview

**KNX-Lens** is a suite of Python tools for monitoring and analyzing KNX bus traffic. It consists of three main components:

1. **`knx-lens-logger.py`**: Daemon that captures live KNX telegrams, decodes them using an ETS project file, and writes rotating logs to disk with gzip compression
2. **`knx-lens.py`**: Interactive TUI (Text-User-Interface) explorer using Textual framework for browsing `.knxproj` files and filtering logs
3. **`knx-lens-web.py`**: Web server wrapper making the TUI accessible via browser (using textual-serve)

All three share common utilities and configuration via `.env` file.

## Architecture & Data Flow

### Three-Tier Data Processing Pipeline

```
KNX Bus (via XKNX lib)
    ↓
knx-lens-logger.py (async telegram capture)
    ↓ [telegram_to_log_message + DPT decoding]
    ↓
knx_bus.log (rotating, daily + ZIP compression)
    ↓
knx_log_utils.py (parse + cache)
    ↓ [detect format: pipe-separated or CSV]
    ↓
knx-lens.py TUI
    ↓ [selection filters + regex + time filters]
    ↓
DataTable display
```

### Key Components & Their Responsibilities

| File | Role |
|------|------|
| `knx-lens.py` | Main TUI app (Textual App + KNXTuiLogic mixin) - 757 lines |
| `knx_tui_logic.py` | Business logic mixin (tree building, filtering, caching) - 557 lines |
| `knx_tui_screens.py` | Modal screens/widgets (FilterInputScreen, FilteredDirectoryTree) |
| `knx_project_utils.py` | ETS `.knxproj` parsing via `xknxproject` library - MD5 caching for performance |
| `knx_log_utils.py` | Log file parsing (pipe-separated + CSV), payload history tracking |
| `knx-lens-logger.py` | Async gateway monitoring (XKNX), rotating log handler with ZIP rotation |
| `setup.py` | Interactive configuration wizard (Textual TUI, gateway auto-discovery) |
| `knx-lens-web.py` | Web server entry point (textual-serve) |

### Project Data Structures

**Loaded from `.knxproj`** (via `xknxproject.XKNXProj`):
- `devices_dict`: Physical address → device info
- `ga_dict`: Group address (GA) → semantic info
- `building_structure`: Hierarchical spaces (floors, rooms, functions)

**In-memory caches** (`knx_tui_logic.py`):
- `cached_log_data`: List[Dict] up to MAX_CACHE_SIZE (50,000 lines)
- `payload_history`: Dict[GA → List[{timestamp, value, previous}]] for "live values in tree" feature
- `project_data["project"]`: Unwrapped `.knxproj` parse result (wrapper has MD5 for cache invalidation)

## Critical Development Workflows

### 1. Setup & Configuration

```bash
python setup.py        # Interactive Textual TUI to configure gateway + paths → .env file
```

Required `.env` variables:
```
KNX_GATEWAY_IP=192.168.x.x     # or "AUTO" to scan on startup
KNX_GATEWAY_PORT=3671
KNX_PROJECT_PATH=/path/to/project.knxproj
LOG_PATH=/path/to/logs/dir
KNX_LOG_FILE=/path/to/logs/knx_bus.log
KNX_PASSWORD=optional_proj_password
```

**Windows quirk**: Use forward slashes in paths to avoid escaping issues.

### 2. Running the Logger

```bash
python knx-lens-logger.py      # Starts async event loop, connects to gateway, logs telegrams
```

- Uses `xknx.XKNX` for KNX/IP communication
- Implements `ZipTimedRotatingFileHandler` for daily rotation → `.zip` compression
- Decodes telegrams to human-readable format using DPT types from project
- Logs format: `timestamp | source_PA | sink_PA | dest_GA | payload_description | raw_payload`

### 3. Running the TUI Explorer

```bash
python knx-lens.py             # Interactive explorer with mouse/keyboard support
python knx-lens.py -v          # Verbose logging to console + knx_debug.log
python knx-lens.py /path/to/log.zip  # Load specific ZIP log file
```

- **Tabs**: "Building" (spaces), "Physical Addresses" (lines), "Group Addresses" (functions), "Selection Groups", "Log View", "Files"
- **Tree selection ('a')**: Toggle nodes → update `selected_gas` set → filter log to matching GAs
- **Selection Groups ('s')**: Save selected GAs + regex rules to `named_filters.yaml` for reuse
- **Global AND filter**: Regex in "Log View" tab applied in addition to selection group filters

### 4. Running the Web Interface

```bash
python knx-lens-web.py         # Starts textual-serve on configured IP:PORT (default 8000)
```

- Auto-detects local IP (with fallback to 127.0.0.1)
- Can be overridden via `WEBSERVER_IP` + `WEBSERVER_PORT` in `.env`
- Multi-user capable; browser tab runs remote Textual session

### 5. Docker Deployment

```bash
docker compose up               # Starts logger + web containers
```

- Logger writes logs to `${DOCKERCONFIG}/knx-lens/logs` (mounted host path)
- Web container mounts same logs as read-only
- `.env` file shared between both services

## Project-Specific Patterns & Conventions

### Log Format Detection (knx_log_utils.py)

```python
def detect_log_format(first_lines: List[str]) -> Optional[str]:
    # Returns "pipe_separated" or "csv" based on heuristics
```

Both formats supported:
- **Pipe-separated** (default from logger): `timestamp | source_PA | dest_PA | group_addr | decode | raw`
- **CSV** (legacy): semicolon-delimited, column indices differ

### Tree Data Structure

All three trees (`building_tree_data`, `pa_tree_data`, `ga_tree_data`) follow pattern:

```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [henfri/knx-lens](https://github.com/henfri/knx-lens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
