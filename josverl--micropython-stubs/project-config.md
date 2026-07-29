---
trigger: always_on
description: This repository mostly contains **type stubs**, not executable code.
---

# MicroPython-Stubs Repository Guide

## Project Overview

This repository mostly contains **type stubs**, not executable code. 
It hosts 3,000+ stub files (`.pyi`) for MicroPython across multiple versions (v1.17-v1.27), ports (esp32, rp2, stm32, etc.), and boards. These stubs enable IDE autocomplete, type checking, and documentation for MicroPython development.

**Sister Repository**: [micropython-stubber](https://github.com/Josverl/micropython-stubber) - CLI tool (`stubber`) that generates these stubs.

## Repository Structure

```
stubs/                  # Generated stub files (MCU, frozen, doc, merged)
publish/                # Ready-to-install PyPI packages
  micropython-v1_XX_X-{port}-{board}-stubs/
  micropython-stdlib-stubs/
tools/                  # Tooling (board_compare)
scripts/                # Analysis scripts and notebooks
docs/                   # Sphinx documentation source
data/                   # JSON database files (stub-packages.json)
tests/quality_tests/    # Pytest-based stub quality tests
```

## Key Concepts

### Stub Types (Half-Products)
1. **MCU stubs**: Generated on-device, precise but minimal type info (`stubs/micropython-{version}-{port}[-{board}]`)
2. **Frozen stubs**: From manifest files, includes frozen modules (`stubs/micropython-{version}-Frozen/{port}/{board}`)
3. **Doc stubs**: Parsed from MicroPython `.rst` docs, rich typing (`stubs/micropython-{version}-docstubs`)
4. **Merged stubs**: MCU + Doc, precise + rich (`stubs/micropython-{version}-{port}-merged`)
5. **Core stubs**: Manual overrides for problematic modules (`stubs/micropython-core`)

### Naming Convention
`micropython-{flat_version}-{port}-{board}-stubs` where `flat_version` uses underscores (e.g., `v1_24_1`).

## Development Workflows

### Platform Notes
- **Primary**: Windows 
- **Secondary**: Linux 
- **CI/CD** :GitHub Actions workflows use Ubuntu

### Database

#### `data/all_packages.db`
The project uses a SQLite database (`package_data.db`) to store metadata about packages 
and project hashes to detect if a package has changed since it was last published.

This database is generated and maintained by the `micropython-stubber` tool during the publishing process.
It is checked in to the repository to allow scripts and tools to query package information as it cannot be 
regenerated easily.

The database can be queried, but should not be modified directly.
There is also a JSON export of the database at `data/all_modules.json` which can be used for read-only queries without needing SQLite.


### Database Access - IMPORTANT
**SQLite3 CLI tools are NOT installed.** When working with `package_data.db` or database queries:
- ✅ **USE**: Data Store MCP servers (available in this environment)
- ❌ **AVOID**: Writing one-off SQLite scripts/commands that require debugging
- Alternative: Query `all_modules.json` (JSON export of package data) or `data/stub-packages.json`

### Core Tools

#### The `stubber` CLI
From [micropython-stubber](https://github.com/Josverl/micropython-stubber), installed via pip:
```powershell
pip install micropython-stubber  # or --pre for preview versions
```

**Common Commands**:
```powershell
stubber clone --add-stubs              # Clone micropython + micropython-lib repos
stubber get-docstubs --version v1.24.1 # Generate doc stubs
stubber get-frozen --version v1.24.1   # Generate frozen stubs
stubber merge --version v1.24.1        # Merge MCU + doc stubs
stubber build --version v1.24.1        # Build publishable package
```

#### Update Scripts
- **`update_all_modules.py`**: Regenerates `all_modules.json` from `publish/` packages
- **`docs/update_docs.py`**: Updates documentation files

#### Build/Test Tasks
VSCode tasks (`.vscode/tasks.json`):
- `Sphinx: build documentation` - Default build (Ctrl+Shift+B)
- `Sphinx: clean build documentation`

### Typical Development Flow

1. **Update stubs for a version** (using stubber CLI):
   ```bash
   # example: version =  v1.24.1
   stubber get-docstubs --version v1.24.1
   stubber get-frozen --version v1.24.1
   stubber merge --version v1.24.1 --port rp2 --board rpi_pico
   stubber build --version v1.24.1 --port rp2 --board rpi_pico
   ```

2. **Interactive development** (Jupyter notebooks):
   - `Manual stub build chain.ipynb` - Interactive stub generation workflow
   - `scripts/package_db_to_json.ipynb` - Database export operations
   - `scripts/find_undoc_funcs.ipynb` - Documentation analysis

3. **Test stub quality**:
   ```bash
   pytest tests/quality_tests/ -v
   # Clear cache and retest
   pytest tests/quality_tests/ --cache-clear
   # Test specific version/port
   pytest tests/quality_tests/ -k "v1_24_1 and rp2"
   ```

4. **Update documentation**:
   ```bash
   uv run docs/update_docs.py
   uv run scripts/update_all_modules.py
   ```

5. **Build docs**:
   Run task: `Sphinx: build documentation`

### GitHub Actions (`.github/workflows/`)
- **`update_stubs.yml`**: Daily stub updates using stubber CLI
- **`test_stub_quality.yml`**: Pyright/mypy validation on snippets (15min runtime)
- **`test_runtime_typing.yml`**: Docker-based runtime tests for typing module
- **`publish_explorer.yml`**: Deploys board explorer app
- **`weekly_automation.yml`**: Maintenance tasks
- **`get-doc-stubs.yml`**: Documentation stub generation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Josverl/micropython-stubs](https://github.com/Josverl/micropython-stubs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
