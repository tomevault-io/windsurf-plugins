---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MDict Utils is a Python library for packing and unpacking MDict dictionary files (.mdx/.mdd). It supports reading/writing MDict Version 2.0 and reading MDict Version 3.0. The project is built using modern Python 3.13+ with uv as the package manager and build backend.

**Major Architecture:** The project uses a modern subcommand architecture with TOML-based metadata management, replacing the old argument-based interface.

## Development Commands

### Package Management
- `uv sync` - Install dependencies and create virtual environment
- `uv add <package>` - Add new dependency
- `uv remove <package>` - Remove dependency

### Code Quality
- `uv run ruff check` - Run linting (pycodestyle, pyflakes, isort, security checks, etc.)
- `uv run ruff format` - Format code
- `uv run pyright` - Run type checking

### Testing
- `tests/run_tests.sh all` - Run all tests using the test runner
- `tests/run_tests.sh unit` - Run unit tests only
- `tests/run_tests.sh integration` - Run integration tests with real files
- `tests/run_tests.sh -c` - Run tests with coverage report
- `uv run pytest` - Direct pytest execution
- `uv run pytest -m "not slow"` - Skip slow tests
- `tests/test_integration.sh` - Run shell-based integration tests

### Running the Tool (New Subcommand Architecture)
- `uv run mdict --help` - Show all available commands
- `uv run mdict extract dict.mdx` - Extract dictionary
- `uv run mdict pack -a source.txt dict.mdx` - Pack dictionary
- `uv run mdict query word dict.mdx` - Query word
- `uv run mdict info dict.mdx` - Show dictionary information
- `uv run mdict keys dict.mdx` - List dictionary keys
- `uv run mdict convert txt-to-db dict.txt dict.db` - Convert formats

## Architecture

### Core Components

1. **Reader Module** (`src/mdict_utils/reader.py`)
   - Handles unpacking .mdx/.mdd files to text/database formats
   - Supports querying individual entries and extracting metadata
   - Uses `base.readmdict.MDX` and `base.readmdict.MDD` classes for low-level file parsing

2. **Writer Module** (`src/mdict_utils/writer.py`)
   - Handles packing text files or databases into .mdx/.mdd format
   - Supports multiple input sources (txt files, SQLite databases, directories)
   - Uses `base.writemdict.MDictWriter` for low-level file writing

3. **Metadata Module** (`src/mdict_utils/metadata.py`) - **NEW**
   - TOML-based metadata management system
   - Automatic metadata file detection and validation
   - Conversion between TOML format and MDX headers
   - MetadataManager class for intelligent caching and operations

4. **Subcommand Architecture** (`src/mdict_utils/commands/`) - **NEW**
   - Modern CLI with individual command modules
   - `extract.py`, `pack.py`, `query.py`, `info.py`, `keys.py`, `convert.py`
   - Each command inherits from `BaseCommand` with consistent interface
   - Rich help system and error handling

5. **Main Entry Point** (`src/mdict_utils/__main__.py`) - **REDESIGNED**
   - Modern argparse with subcommands (like git, docker, etc.)
   - Global options (--verbose, --quiet) and per-command options
   - Comprehensive help and usage examples
   - Graceful error handling with user-friendly messages

6. **Base Classes** (`src/mdict_utils/base/`)
   - Low-level MDict file format implementation
   - Handles compression, block structures, and binary format details

### File Format Support

- **MDX files**: Dictionary entries (word definitions)
- **MDD files**: Media resources (images, audio, etc.)
- **Database format**: SQLite3 with `mdx` and `mdd` tables for faster access
- **Text format**: UTF-8 encoded key-value pairs

### Key Features

- **TOML Metadata Management**: User-friendly `.meta.toml` files for dictionary metadata
- **Subcommand Architecture**: Modern CLI with `extract`, `pack`, `query`, `info`, `keys`, `convert` commands
- **Automatic Metadata Detection**: Finds `.meta.toml` files automatically based on source file names
- **Multiple Output Formats**: Text, database, split files, JSON, TOML
- **Rich Information Display**: Beautiful formatted output with color and structure
- **Progress Tracking**: Visual progress bars for long operations
- **Support for MDict Versions**: 1.2, 2.0, 3.0 with automatic version detection
- **Advanced Features**: Encrypted dictionaries, substyle support, compact HTML conversion
- **Comprehensive Testing**: Unit, integration, and shell-based tests with real dictionary files

## Testing Structure

Tests are organized in the `tests/` directory:
- `test_packing.py` - Legacy packing tests
- `test_unpacking.py` - Legacy unpacking tests  
- `test_subcommands.py` - **NEW** Tests for subcommand architecture
- `test_metadata.py` - **NEW** Tests for TOML metadata functionality
- `conftest.py` - **NEW** Pytest configuration and shared fixtures
- `test_config.toml` - **NEW** Configuration for real dictionary test files
- `run_tests.sh` - **NEW** Smart test runner with multiple execution modes
- `test_integration.sh` - **NEW** Shell-based integration testing
- `README.md` - **NEW** Comprehensive testing documentation

### Test Categories and Markers:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [libukai/mdtt](https://github.com/libukai/mdtt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
