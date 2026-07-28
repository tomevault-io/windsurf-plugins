---
trigger: always_on
description: Surfactant is a modular framework for gathering file information to generate Software Bill of Materials (SBOM) and perform dependency analysis. It extracts metadata from various file types (PE, ELF, MSI, etc.) without requiring file execution or decompilation.
---

# Surfactant - Copilot Instructions

## Project Overview

Surfactant is a modular framework for gathering file information to generate Software Bill of Materials (SBOM) and perform dependency analysis. It extracts metadata from various file types (PE, ELF, MSI, etc.) without requiring file execution or decompilation.

## Key Technologies

- **Language**: Python 3.10+
- **CLI Framework**: Click 8.x
- **TUI Framework**: Textual 6.x
- **Plugin System**: Pluggy 1.x
- **SBOM Formats**: SPDX (spdx-tools 0.8.x), CycloneDX (cyclonedx-python-lib 11.5.0)
- **File Parsing**: pyelftools, pefile, dnfile, olefile, rarfile, python-msi, rpmfile
- **Build System**: setuptools with setuptools-scm for versioning
- **Testing**: pytest with pytest-asyncio
- **Linting**: ruff (linter + formatter) and pylint
- **Pre-commit**: Configured with shellcheck and other hooks

## Directory Structure

```
surfactant/              # Main package
├── cmd/                 # CLI commands (generate, merge, cli, tui, etc.)
├── configmanager.py     # Configuration management
├── context.py           # SBOM context handling
├── database_manager/    # Database utilities
├── fileinfo.py          # File information structures
├── filetypeid/          # File type identification
├── infoextractors/      # Metadata extraction per file type
├── input_readers/       # SBOM readers (SPDX, CycloneDX, JSON)
├── output/              # SBOM writers
├── plugin/              # Plugin system implementation
├── relationships/       # Dependency relationship handling
├── sbomtypes/           # SBOM data structures
└── utils/               # Utility functions

plugins/                 # Optional plugins (separate packages)
├── fuzzyhashes/         # Fuzzy hashing plugin
├── grype/               # Grype vulnerability scanner integration
├── syft/                # Syft SBOM tool integration
└── ...                  # Other plugins

tests/                   # Test suite
├── cmd/                 # CLI command tests
├── config/              # Configuration tests
├── data/                # Test data
└── ...                  # More test modules

docs/                    # Sphinx documentation
example-configs/         # Example configuration files
scripts/                 # Development and utility scripts
```

## Development Setup

### Installation

```bash
# Create and activate virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install in editable mode with dev dependencies
pip install -e ".[test,dev]"

# Install optional dependencies for specific file formats
pip install -e ".[macho,java]"

# Install plugins for development
pip install -e plugins/fuzzyhashes
```

### Running Tests

```bash
# Run all tests
pytest

# Run specific test file
pytest tests/cmd/test_cli.py

# Run with verbose output
pytest -v

# Collect tests without running
pytest --co
```

### Code Quality

```bash
# Run pre-commit hooks
pre-commit run --all-files

# Run ruff linter
ruff check .

# Run ruff formatter
ruff format .

# Run pylint
pylint surfactant
```

## Code Style Guidelines

### Formatting

- **Line Length**: 100 characters (configured in pyproject.toml)
- **Indent**: 4 spaces
- **Formatter**: ruff-format (automatic via pre-commit)

### Linting Rules

- **Ruff**: Enforces E (pycodestyle errors), F (pyflakes), B (flake8-bugbear), I (isort)
- **Exceptions**:
  - E501 (line too long) - ignored, handled by formatter
  - F841 (unused variable) - ignored
  - B (bugbear violations) - not auto-fixed
- **Pylint**: Most docstring and complexity checks disabled to reduce noise

### Naming Conventions

- **Good short names**: x, y, e, md, sw (as per pylint config)
- **Functions/Variables**: snake_case
- **Classes**: PascalCase
- **Constants**: UPPER_SNAKE_CASE

## Important Patterns and Conventions

### Plugin System

Surfactant uses a pluggy-based plugin system. Plugins can:
- Extract additional file information
- Add custom SBOM fields
- Integrate external tools (grype, syft, etc.)

Plugins are separate packages installed via pip/pipx.

### SBOM Generation Workflow

1. **File Identification**: Detect file types using magic numbers and extensions
2. **Information Extraction**: Extract metadata using specialized extractors
3. **Relationship Detection**: Identify dependencies between components
4. **SBOM Output**: Write to SPDX or CycloneDX format

### Configuration

- Managed via `ConfigManager` singleton
- Settings stored in TOML format
- Can be set via CLI: `surfactant config set <key> <value>`

### Context Objects

The `ContextEntry` class is central to SBOM generation, containing:
- Detailing what directories should be traversed
- Configuring what file types should be included/omitted from the SBOM
- Contextual information on where the files in the traversed directory would be if installed on a "real" system

## Testing Guidelines

- Place tests in `tests/` directory mirroring the source structure
- Use pytest fixtures for common setup
- Test data in `tests/data/`
- Use `pytest-asyncio` for async tests
- Tests should not modify global state

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LLNL/Surfactant](https://github.com/LLNL/Surfactant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
