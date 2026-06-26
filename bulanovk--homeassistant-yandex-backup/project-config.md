---
trigger: always_on
description: **Project Name**: Yandex Disk Backup Provider (ha-yandex-location)
---

# Project Memory: Yandex Disk Backup Provider for Home Assistant Core

## Project Overview

**Project Name**: Yandex Disk Backup Provider (ha-yandex-location)
**Goal**: Implement a Home Assistant Core integration for Yandex Disk as a backup storage provider
**Status**: Implementation Complete - Ready for testing and contribution

## Technical Requirements

### Python Environment
- **MUST use virtual environment** for Python development and testing
- Create venv: `python -m venv venv`
- Activate venv:
  - Windows: `venv\Scripts\activate`
  - Linux/Mac: `source venv/bin/activate`
- Install dependencies in venv only

### Primary Dependency
- `yadisk>=3.4.0` - Official Python async client for Yandex Disk API

### Home Assistant Requirements
- Minimum HA Version: 2023.8.0 (backup agent platform support)
- Target HA Versions: 2024.x - 2025.x
- Tested with: homeassistant==2025.12.4
- Python: 3.13+ (recommended), 3.12+ supported

## Implementation Plan Location

The detailed implementation plan is saved at:
`C:\Users\bulan\.claude\plans\humble-crunching-dahl.md`

This plan contains:
1. Architectural Overview
2. Integration Classification (domain: `yadisk`, Backup Provider)
3. Authentication & Credential Handling (OAuth token-based)
4. Async Design (yadisk.AsyncClient, no executor needed)
5. Backup Lifecycle (all 5 BackupAgent methods)
6. Configuration Flow & UX
7. Failure Handling & Resilience
8. Testing Strategy
9. Dependency Management
10. Logging & Diagnostics
11. Risks & Reviewer Concerns

## File Structure

```
d:/projects/ha-yandex-location/
├── custom_components/
│   └── yandex_disk_backup/
│       ├── __init__.py         # Component setup, agent registration
│       ├── backup.py           # YandexDiskBackupAgent (core logic)
│       ├── config_flow.py      # OAuth token validation
│       ├── const.py            # Constants (DOMAIN, config keys)
│       ├── diagnostics.py      # Diagnostic endpoint
│       ├── manifest.json       # Component metadata, deps
│       ├── strings.json        # UI strings
│       └── translations/
│           └── en.json         # English translations
├── tests/
│   ├── conftest.py             # Windows compatibility mocks
│   └── custom_components/
│       └── yandex_disk_backup/
│           ├── __init__.py
│           ├── conftest.py     # Pytest fixtures
│           ├── test_backup.py  # Backup operations tests
│           └── test_config_flow.py # Config flow tests
├── .github/
│   └── workflows/
│       ├── ci.yaml             # GitHub Actions CI (Python 3.13)
│       └── hacs.off            # HACS validation workflow
├── requirements.txt            # Production dependencies
├── requirements_test.txt       # Test dependencies
├── README.md                   # Documentation
└── CLAUDE.md                   # This memory file
```

## Implementation Status

### Completed Files (16)

1. **const.py** - Domain: "yandex_disk_backup", config keys, timeouts, defaults
2. **manifest.json** - Dependencies: yadisk>=3.4.0, backup platform
3. **backup.py** - YandexDiskBackupAgent class with:
   - async_upload_backup() - with space check, verification, descriptive filenames, metadata sidecar upload
   - async_download_backup() - streaming with 4MB chunks via HA's HTTP session
   - async_delete_backup() - trash (not permanent delete), also removes metadata sidecar
   - async_list_backups() - filtered by .tar/.tar.gz extension, sorted newest first, loads metadata from sidecar
   - async_get_backup() - metadata retrieval from sidecar or fallback to file metadata
   - _ensure_backup_folder() - creates folder if missing
   - _get_disk_info_cached() - 5-minute cache
   - _get_client() - client creation wrapped in executor to avoid SSL blocking
   - _get_metadata_path() - generates sidecar metadata file path
   - _upload_metadata() - stores backup metadata as JSON sidecar file
   - _load_metadata() - loads backup metadata from sidecar file
4. **config_flow.py** - OAuth token validation via get_disk_info()
5. **__init__.py** - async_setup_entry, async_unload_entry, agent registration
6. **strings.json** - UI strings for config flow
7. **diagnostics.py** - async_get_config_entry_diagnostics with token redaction
8. **translations/en.json** - English translations
9. **tests/conftest.py** - Windows compatibility mocks (fcntl, resource)
10. **tests/custom_components/yandex_disk_backup/__init__.py** - Test package marker
11. **tests/custom_components/yandex_disk_backup/conftest.py** - Mock fixtures
12. **tests/custom_components/yandex_disk_backup/test_backup.py** - 14 test cases
13. **tests/custom_components/yandex_disk_backup/test_config_flow.py** - 9 test cases
14. **README.md** - Documentation
15. **.github/workflows/ci.yaml** - CI configuration (Python 3.13)
16. **.github/workflows/hacs.off** - HACS validation workflow

## Key Design Decisions

1. **Domain Name**: `yandex_disk_backup` (custom component, uses yadisk package)
2. **Classification**: Backup Provider only (not storage provider)
3. **Authentication**: OAuth token-based (not full OAuth flow)
4. **Async Strategy**: Native yadisk.AsyncClient, with client creation in executor to avoid SSL blocking
5. **Upload Strategy**: Uses `client.upload()` with User-Agent spoofing to bypass 128 KiB/s throttling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bulanovk/homeassistant-yandex-backup](https://github.com/bulanovk/homeassistant-yandex-backup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
