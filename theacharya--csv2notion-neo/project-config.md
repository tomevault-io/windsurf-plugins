---
trigger: always_on
description: This is CSV2Notion Neo, an advanced command-line tool for uploading and merging CSV or JSON files with images to Notion databases. The project is written in Python 3.11+ and uses Poetry for dependency management.
---

# CSV2Notion Neo - Cursor Rules

## Project Context

This is CSV2Notion Neo, an advanced command-line tool for uploading and merging CSV or JSON files with images to Notion databases. The project is written in Python 3.11+ and uses Poetry for dependency management.

## Version 2.0.0+ Migration

CSV2Notion Neo has been fully migrated to use the official [Notion API](https://developers.notion.com/) (version 2025-09-03) and the [notion-sdk-py](https://github.com/ramnes/notion-sdk-py) library (`notion-client` ^3.1.0). This migration provides better reliability, security, and future compatibility. The application now uses Notion integration tokens instead of session cookies and requires database URLs to be provided for all operations.

### Migration Implementation Details

#### New File Structure
- `csv2notion_neo/notion_client.py`: Notion API client wrapper with comprehensive retry logic
- `csv2notion_neo/notion_db.py`: Database operations with enhanced schema detection and thread-safe select option management
- `csv2notion_neo/notion_row_upload_file.py`: File upload operations with retry logic
- `csv2notion_neo/notion_db_client.py`: Extended client with additional functionality

#### Removed Components
- `csv2notion_neo/notion/` directory: Legacy API implementation
- `csv2notion_neo/notion_row.py`: Old row handling classes
- `csv2notion_neo/notion_db_collection.py`: Old collection management
- `csv2notion_neo/notion_client_official.py`: Renamed to `notion_client.py`
- `csv2notion_neo/notion_db_official.py`: Renamed to `notion_db.py`
- `csv2notion_neo/notion_row_upload_file_official.py`: Renamed to `notion_row_upload_file.py`
- `csv2notion_neo/notion_db_client_official.py`: Renamed to `notion_db_client.py`

#### Key Implementation Changes
- Authentication: Integration tokens with format validation instead of session cookies
- API Version: Notion API 2025-09-03 with data_sources structure for database properties
- Database Operations: Notion databases API endpoints with page URL support for creating databases within existing pages
- Data Sources: Database properties retrieved from data_sources endpoint instead of database object
- Schema Updates: Database schema modifications use data_sources.update endpoint
- Database Creation: Uses initial_data_source for new database property definitions
- File Uploads: Notion file_uploads API endpoints with comprehensive retry logic
- Files Property Support: `--image-column-mode file` maps values to Notion `files` property items (`file_upload` for local files, `external` for URLs)
- Error Handling: Notion API error codes and responses with smart retry strategies
- Rate Limiting: Built-in API rate limiting with exponential backoff; HTTP 429 handling with Retry-After parsing and cross-thread ban coordination; proactive throttle (~3 req/s) to reduce 429s (issue #76)
- Property Validation: API property type validation with intelligent schema detection
- Thread Safety: Thread-safe select option management to prevent race conditions during concurrent uploads
- Retry Logic: Enhanced retry system with 15 attempts, exponential backoff, and jitter for production reliability
- Logic Path Separation: Clear distinction between creating new databases in pages vs uploading to existing databases
- Token Validation: User-friendly token format validation with clear error messages

#### Compatibility Layer
- Maintains existing method signatures where possible
- Provides seamless transition from old to new API
- Handles data structure differences between APIs
- Preserves all existing functionality and features

## Dependencies

### Runtime (`[tool.poetry.dependencies]`)
- Python `>=3.11,<3.15` (local builds and CI use **3.14.5** via `scripts/local-test-build.sh`)
- `notion-client` ^3.1.0 (notion-sdk-py), `requests`, `tqdm`, `emoji`, `python-dateutil`, `icecream`
- `icecream`: debug helper aligned with Airlift (`ic.enable()` in `cli.py`; `ic(...)` on thread-pool errors in `utils_threading.py`)

### Dev-only (`[tool.poetry.group.dev.dependencies]`)
- Testing: `pytest` ^9, `pytest-cov` ^7, `pytest-mock`, `pyfakefs` ^6
- Lint/format: `black` ^26, `isort` ^8, `flake8` ^7.3, `wemake-python-styleguide` ^1.6, `mypy` ^2
- Types: `types-requests`, `types-python-dateutil`, `types-emoji`
- `python-dotenv` ^1 — tests only (`tests/input_command.py` for integration test env)
- `pre-commit` ^4, `mdformat` ^1

### Removed legacy runtime deps (no longer in `pyproject.toml`)
- Unused in codebase: `ratelimit`, `commonmark`, `dictdiffer`, `python-slugify`, `bs4`, `cached-property`, `tzlocal` (rate limiting is implemented in `notion_client.py`)
- Dev-only now: `python-dotenv`
- Removed dev deps: `flakehell` (incompatible with flake8 7+), `pytest-vcr`, `testfixtures`

## Code Style and Standards

### Python Code Style
- Use Black formatting with 88 character line length
- Follow PEP 8 guidelines
- Use type hints throughout the codebase
- Use dataclasses for configuration objects
- Implement proper error handling with custom exceptions

### Import Organization
- Use isort for import organization
- Group imports: standard library, third-party, local

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheAcharya/csv2notion-neo](https://github.com/TheAcharya/csv2notion-neo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
