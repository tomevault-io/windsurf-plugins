---
trigger: always_on
description: This is a containerized iCloud sync client that downloads files/photos from iCloud Drive and Photos to local filesystem. Built as a long-running Docker service using Alpine Linux with custom process management (no S6 overlay - uses `su-exec` and `docker-entrypoint.sh`).
---

# iCloud Docker - AI Coding Agent Instructions

## Project Overview
This is a containerized iCloud sync client that downloads files/photos from iCloud Drive and Photos to local filesystem. Built as a long-running Docker service using Alpine Linux with custom process management (no S6 overlay - uses `su-exec` and `docker-entrypoint.sh`).

## Architecture & Core Components

### Main Sync Loop (`src/sync.py`) - **RECENTLY REFACTORED**
- **Orchestrator pattern**: Main `sync()` function delegates to 15+ focused helper functions following SRP
- **State management**: `SyncState` class encapsulates countdown timers and sync flags to avoid parameter passing
- **Adaptive scheduling**: `_calculate_next_sync_schedule()` determines which service syncs next based on timers
  - Tracks `drive_time_remaining` and `photos_time_remaining` in state object
  - Subtracts elapsed time from other service's timer when one service syncs
- **Oneshot mode**: Set sync_interval to `-1` to run once and exit (useful for cron-style scheduling)
  - Exit condition: `_should_exit_oneshot_mode()` checks if ALL configured intervals are negative
- **2FA handling**: `_handle_2fa_required()` manages authentication retry with configurable intervals
- **Password management**: `_retrieve_password()` uses keyring storage, falls back to `ENV_ICLOUD_PASSWORD`
  - Stores password from env var into keyring on first run for persistence

### Configuration System - **HEAVILY REFACTORED**
- **Layered architecture**: Core utilities in `config_utils.py`, logging in `config_logging.py`, main API in `config_parser.py`
- **YAML-based** with deep path traversal: `traverse_config_path()` → `get_config_value()` pattern (NEVER direct dict access)
- **Runtime config reloading**: Config re-read on every sync loop iteration via `_load_configuration()`
- **Environment overrides**: `ENV_CONFIG_FILE_PATH` and `ENV_ICLOUD_PASSWORD` override file values
- **Validation pattern**: ALWAYS use `traverse_config_path()` before `get_config_value()` to avoid KeyError
- **Thread configuration**: `get_app_max_threads()` supports `"auto"` (caps at 8) or integer 1-16
- **Zero duplication**: Shared patterns extracted into helper functions (e.g., `get_sync_interval()`)

### Drive Sync Modules - **SPLIT INTO 8 SPECIALIZED MODULES**
- **`sync_drive.py`**: High-level orchestration and folder processing
- **`drive_parallel_download.py`**: Parallel download coordination with `ThreadPoolExecutor`
- **`drive_file_download.py`**: Individual file download and atomic operations
- **`drive_filtering.py`**: File/folder filtering logic via glob matching
- **`drive_file_existence.py`**: File existence checks and package detection
- **`drive_cleanup.py`**: Obsolete file removal when `remove_obsolete` enabled
- **`drive_package_processing.py`**: ZIP auto-extraction and gzip handling with `magic` library
- **`drive_folder_processing.py`**: Directory traversal and recursive processing

### Photos Sync Modules - **SPLIT INTO 7 SPECIALIZED MODULES**
- **`sync_photos.py`**: High-level orchestration (libraries vs albums)
- **`album_sync_orchestrator.py`**: Album synchronization coordination
- **`photo_download_manager.py`**: Parallel download task collection and execution
- **`photo_filter_utils.py`**: Photo filtering by extensions and album preferences
- **`photo_path_utils.py`**: Path normalization and folder format handling
- **`photo_file_utils.py`**: File operations and metadata handling
- **`hardlink_registry.py`**: `HardlinkRegistry` class for deduplication across albums
- **File sizes**: `original`, `original_alt` (RAW fallback), `medium`, `thumb`
- **Hardlink deduplication**: `use_hardlinks` mode with registry tracking across albums
- **Date organization**: `folder_format` uses strftime patterns (e.g., `"%Y/%m"`)

### Notification System (`src/notify.py`)
- **Multi-provider**: Discord, Telegram, Pushover, SMTP with rate limiting (24-hour throttle)
- **2FA alerts**: Automatically notifies when authentication expires (`api.requires_2sa`)
- **Rate limiting**: `last_send` parameter prevents notification spam (returns same timestamp if < 24hrs)

### Usage Tracking (`src/usage.py`)
- **Opt-in telemetry**: Collects anonymized sync statistics for usage analytics (see `USAGE.md`)
- **Opt-out**: Set `app.usage_tracking.enabled: false` in config to disable completely
- **Data collected**: Version, sync duration/counts, error indicators (no file names/paths/credentials)
- **Endpoints**: `NEW_INSTALLATION_ENDPOINT` and `NEW_HEARTBEAT_ENDPOINT` from Dockerfile build args

## Development Workflow

### Local Testing
```bash
# Run full CI pipeline locally (includes ruff, pytest, allure report)
source .venv/bin/activate && ./run-ci.sh
# Manually: ruff check --fix && ENV_CONFIG_FILE_PATH=./tests/data/test_config.yaml pytest
```

### Key Testing Patterns
- **Mock strategy**: Tests use `ICloudPyServiceMock` in `tests/data/__init__.py` (4000+ lines) with photo fixtures in `tests/data/photos_data.py` (2800+ lines)
- **Config injection**: Tests override config paths via `tests.CONFIG_PATH` and `tests.TEMP_DIR`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mandarons/icloud-docker](https://github.com/mandarons/icloud-docker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
