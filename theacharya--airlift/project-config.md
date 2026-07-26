---
trigger: always_on
description: This file provides comprehensive guidance for AI agents (Claude, Cursor, etc.) when working with code in this repository.
---

# Airlift - AI Agent Documentation

This file provides comprehensive guidance for AI agents (Claude, Cursor, etc.) when working with code in this repository.

## Project Overview

Airlift is a Python-based command-line tool for uploading and merging CSV or JSON data files with attachments to Airtable databases. It uses Dropbox as a temporary storage provider for attachments since Airtable's API doesn't support direct file uploads. The project is built with Poetry for dependency management and uses PyInstaller for cross-platform binary distribution.

## Quick Reference Commands

### Building and Testing
```bash
# Build the project using the ephemeral build system (recommended)
./scripts/local-test-build.sh

# Test the built binary
./test-build/airlift --help

# Clean build environment
./scripts/local-test-build.sh --clean

# Build with dependency updates
./scripts/local-test-build.sh --update-deps

# Run comprehensive tests (no API tokens required)
./scripts/local-test-build.sh --comprehensive-test

# Dev CLI (after --comprehensive-test or full build; uses .build/venv/)
.build/venv/bin/airlift --help

# Prefer the binary for release parity
./test-build/airlift --help
```

### Dependency Management
```bash
# Regenerate poetry.lock (after pyproject.toml changes)
./scripts/local-test-build.sh --lock-only

# Update one package at a time (recommended; updates lock file)
./scripts/local-test-build.sh --lock-only --update requests
./scripts/local-test-build.sh --comprehensive-test

# Update lock then full build
./scripts/local-test-build.sh --update requests

# Update all deps within ^ ranges (use sparingly)
./scripts/local-test-build.sh --update-deps

# Show outdated packages (uses .build/ Poetry)
./scripts/local-test-build.sh --show-outdated
```

Prefer `./scripts/local-test-build.sh` over bare `poetry lock` so resolution uses the
ephemeral Python 3.14.5 environment in `.build/`.

## Architecture Overview

### Core Components

The project follows a modular architecture with clear separation of concerns:

| Module | Description |
|--------|-------------|
| `cli.py` + `cli_args.py` | Command-line interface and argument parsing |
| `airtable_client.py` + `airtable_upload.py` | Airtable API integration using pyairtable 3.x |
| `dropbox_client.py` | Dropbox API integration for file storage using SDK 12.x |
| `csv_data.py` + `json_data.py` | Data file parsing and validation |
| `utils_exceptions.py` | Shared custom exception hierarchy |

### Data Flow Architecture

1. **Input Processing**: CSV/JSON files are parsed and validated
2. **Schema Validation**: Airtable table schema is fetched and columns are mapped
3. **Attachment Handling**: Files are uploaded to Dropbox and sharing URLs generated
4. **Concurrent Upload**: Data is uploaded to Airtable using ThreadPoolExecutor
5. **Progress Tracking**: Real-time progress bars and comprehensive logging

### Key Design Patterns

- **Modular Architecture**: Clear separation between data processing, API clients, and CLI
- **Error Handling**: Custom exception hierarchy with proper error propagation
- **Concurrent Processing**: ThreadPoolExecutor for parallel uploads with configurable workers
- **API Integration**: RESTful clients for Airtable and Dropbox with proper authentication

## Key Features

### Data Format Support
- CSV files with UTF-8 encoding
- JSON files with array of objects structure
- Automatic column validation and mapping
- Support for duplicate column handling

### Airtable Integration
- Personal access token authentication
- Base and table ID validation
- Automatic column creation (configurable)
- Support for single/multiple select fields
- Column renaming and copying capabilities
- Delete all database entries functionality

### Dropbox Operations
- Empty folder contents without deleting the folder itself
- Works with both `/Airlift` and `/Marker Data` folders (via `--md` flag)
- Progress bar for deletion operations

### Attachment Handling
- Dropbox integration for file storage
- Multiple attachment column support
- Column mapping for attachment fields
- Automatic file path resolution

### Performance Features
- Multi-threaded upload processing
- Configurable worker thread count
- Progress bar with real-time updates
- Comprehensive logging system

## Code Organization

### Module Structure
- Keep each module focused on a single responsibility
- Use clear, descriptive module names that reflect their purpose
- Maintain consistent import patterns across modules
- Avoid circular dependencies between modules

### File Naming Conventions
- Use snake_case for all Python files and functions
- Use descriptive names that clearly indicate functionality
- Follow the existing naming pattern: `airlift_*.py` for core modules

## Important Development Guidelines

### Build System Requirements
- **ALWAYS** use `./scripts/local-test-build.sh` for building and dependency lock updates
- The build script is **fully self-contained** under `.build/` (no system Python/Poetry install)
- Downloads **CPython 3.14.5** via [python-build-standalone](https://github.com/astral-sh/python-build-standalone/releases)
- PyInstaller is installed separately during build (not in pyproject.toml)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheAcharya/Airlift](https://github.com/TheAcharya/Airlift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
