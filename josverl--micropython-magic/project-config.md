---
trigger: always_on
description: This repository contains **micropython-magic**, a Python package that provides Jupyter magic commands for executing MicroPython code directly in Jupyter notebooks and JupyterLab. The magic commands enable seamless integration between host Python environments and MicroPython-enabled microcontrollers (MCUs).
---

# Copilot Instructions for micropython-magic

## Repository Overview

This repository contains **micropython-magic**, a Python package that provides Jupyter magic commands for executing MicroPython code directly in Jupyter notebooks and JupyterLab. The magic commands enable seamless integration between host Python environments and MicroPython-enabled microcontrollers (MCUs).

### Key Features
- Execute MicroPython code on connected MCUs from Jupyter cells
- Mix host Python and MCU MicroPython code in the same notebook
- Real-time data visualization and sensor monitoring
- Memory usage analysis and MCU diagnostics
- Support for multiple MCU platforms (ESP32, Raspberry Pi Pico, etc.)
- WOKWI simulator integration for testing without hardware

## Project Structure

```
├── src/micropython_magic/          # Main package source code
│   ├── __init__.py                 # Package initialization and IPython extension loading
│   ├── octarine.py                 # Core MicroPythonMagic class with cell/line magics
│   ├── mpr.py                      # MPRemote2 wrapper for MCU communication
│   ├── interactive.py              # Interactive session management
│   ├── logger.py                   # Logging configuration and MCU log patching
│   ├── magic_transformer.py        # Input transformers for comment magics
│   ├── memoryinfo.py              # Memory analysis utilities
│   ├── param_fixup.py             # Parameter processing utilities
│   ├── script_access.py           # Script execution helpers
│   └── scripts/                   # Helper scripts for MCU operations
├── samples/                       # Example Jupyter notebooks demonstrating features
├── tests/                         # Test suite
│   ├── testbook_cases/           # Full notebook tests
│   ├── testbook_coded/           # Python test files with corresponding notebooks
│   └── *.py                      # Unit tests
├── docs/                         # Documentation assets
├── pyproject.toml               # Poetry configuration and project metadata
└── readme.md                   # Main documentation
```

## Core Architecture

### Magic Commands
The package implements two main types of Jupyter magic commands:

1. **Cell Magic (`%%micropython`)**: Executes entire cell content on the MCU
2. **Line Magic (`%micropython`)**: Executes single line commands or configuration

### Key Classes

- **`MicroPythonMagic`** (`octarine.py`): Main magic class inheriting from IPython's `Magics`
  - Implements `@cell_magic` and `@line_magic` decorators
  - Handles MCU connection management and code execution
  - Provides configuration options (timeout, logging level)

- **`MPRemote2`** (`mpr.py`): Wrapper around mpremote tool
  - Manages serial communication with MCUs
  - Handles code execution, file transfers, and MCU control
  - Provides error handling and response parsing

### Input Transformers
- **`comment_magic_transformer`**: Allows magic commands in comments (e.g., `# %%micropython`)
- Registered during IPython extension loading

## Development Setup

### Dependencies
- **Runtime**: Python ≥3.8, mpremote, loguru, ipympl
- **Development**: Poetry, pytest, testbook, black, pylance
- **Optional**: matplotlib, bqplot, numpy (for visualization features)

### Installation Methods

1. **Using Poetry** (recommended for development):
   ```bash
   poetry install
   poetry install --extras widgets  # For visualization features
   ```

2. **Using pip** (for users):
   ```bash
   pip install micropython-magic
   pip install micropython-magic[widgets]  # With visualization
   ```

### Code Style
- **Formatter**: Black with 100-character line length
- **Type Hints**: Used throughout the codebase
- **Logging**: Uses loguru with structured logging patterns

## Testing Strategy

### Test Framework
The project uses **pytest** with **testbook** for notebook testing:

- **`testbook`**: Tests Jupyter notebooks by executing cells programmatically
- **MCU Requirements**: Most tests require physical MCU hardware
- **No CI**: Tests can't run in GitHub Actions due to hardware dependencies

### Test Categories

1. **Unit Tests** (`tests/*.py`): Traditional pytest tests for utility functions
2. **Notebook Tests** (`test_samples.py`): Validates all sample notebooks
3. **Testbook Cases** (`tests/testbook_cases/`): Full notebook test scenarios
4. **Testbook Coded** (`tests/testbook_coded/`): Python test files with paired notebooks

### Running Tests
```bash
pytest tests/                    # Run all tests
pytest tests/test_samples.py     # Test sample notebooks
pytest --cov=src/micropython_magic  # With coverage
```

## Key Concepts for AI Assistance

### MicroPython Integration
- **mpremote**: Command-line tool for MicroPython MCU communication
- **Serial Protocol**: Uses serial/USB communication with MCUs
- **Code Execution**: Sends Python code as strings to MCU REPL
- **Error Handling**: Parses MCU tracebacks and presents them in Jupyter

### Jupyter Magic System
- **IPython Extension**: Loaded via `%load_ext micropython_magic`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Josverl/micropython-magic](https://github.com/Josverl/micropython-magic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
