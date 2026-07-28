---
trigger: always_on
description: This document provides comprehensive guidance for AI coding agents working on the DataLab codebase. It covers architecture patterns, development workflows, and project-specific conventions.
---

# DataLab AI Coding Agent Instructions

This document provides comprehensive guidance for AI coding agents working on the DataLab codebase. It covers architecture patterns, development workflows, and project-specific conventions.

## Project Overview

**DataLab** is an open-source platform for scientific and technical data processing with a Qt-based GUI. It processes **signals** (1D curves) and **images** (2D arrays).

### Key Architecture Components

1. **Sigima**: Separate computation library providing all processing algorithms (`sigima.proc`)
2. **DataLab GUI**: Qt application layer built on PlotPyStack (PlotPy + guidata)
3. **Processor Pattern**: Bridge between GUI and computation functions
4. **Action Handler**: Manages menus, toolbars, and GUI actions
5. **Plugin System**: Extensible architecture for third-party features (with hot-reload)
6. **Macro System**: User-scriptable automation via Python
7. **Remote Control**: XML-RPC API for external applications
8. **Web API**: FastAPI-based HTTP/JSON server for notebook integration and remote control

### Technology Stack

- **Python**: 3.9+ (using `from __future__ import annotations`)
- **Core Libraries**: NumPy (>=1.22), SciPy (>=1.10.1), scikit-image (>=0.19.2), OpenCV (optional)
- **GUI**: Qt via PlotPy (>=2.8.2) and guidata (>=3.13.4)
- **Computation**: Sigima (>=1.1.2) - separate package
- **Web API**: FastAPI (>=0.110.0), uvicorn (>=0.27.0), pydantic (>=2.0)
- **Other**: pandas (>=1.4), PyWavelets (>=1.2), psutil (>=5.8), packaging (>=21.3)
- **Testing**: pytest with coverage
- **Linting/Formatting**: Ruff (preferred), Pylint (with specific disables)
- **Internationalization**: gettext (.po files), sphinx-intl for docs
- **Documentation**: Sphinx with French translations
- **Packaging**: PyInstaller (standalone), WiX (MSI installer)

### Workspace Structure

```
DataLab/
+-- datalab/              # Main application code
│   +-- gui/              # GUI layer
│   │   +-- processor/    # Processor pattern (signal.py, image.py, base.py)
│   │   +-- actionhandler.py  # Menu/action management
│   │   +-- main.py       # Main window
│   │   +-- panel/        # Signal/Image panels
│   +-- control/          # Remote control API
│   │   +-- baseproxy.py  # Abstract API definition (BaseProxy)
│   │   +-- proxy.py      # RemoteProxy, LocalProxy
│   │   +-- remote.py     # XML-RPC server
│   +-- webapi/           # Web API (FastAPI HTTP/JSON server)
│   │   +-- routes.py     # API endpoint definitions
│   │   +-- controller.py # Business logic
│   │   +-- adapter.py    # DataLab GUI adapter
│   │   +-- schema.py     # Pydantic models
│   │   +-- serialization.py  # NPZ data serialization
│   │   +-- actions.py    # GUI actions (start/stop server)
│   +-- adapters_plotpy/  # PlotPy integration adapters
│   +-- adapters_metadata/  # Metadata adapters
│   +-- h5/               # HDF5 I/O layer
│   +-- widgets/          # Qt widgets (dialogs, editors, viewers)
│   +-- utils/            # Utilities (instance check, etc.)
│   +-- plugins/          # Built-in plugins
│   +-- plugins.py        # Plugin system implementation
│   +-- tests/            # pytest test suite
│   +-- locale/           # Translations (.po files)
│   +-- config.py         # Configuration management
│   +-- objectmodel.py    # Object data model for GUI
+-- doc/                  # Sphinx documentation
│   +-- locale/fr/        # French documentation translations
│   +-- features/         # Feature documentation (signal/, image/)
+-- macros/examples/      # Demo macros
+-- scripts/              # Build/development scripts
│   +-- run_with_env.py   # Environment loader (.env support)
+-- .env                  # Local Python path (PYTHONPATH=.;../guidata;../plotpy;../sigima)
+-- pyproject.toml        # Project configuration
```

**Related Projects** (sibling directories in multi-root workspace):
- `../Sigima/` - Computation library
- `../PlotPy/` - Plotting library
- `../guidata/` - GUI toolkit
- `../PythonQwt/` - Qwt bindings

## Development Workflows

### Running Commands

**ALWAYS use `scripts/run_with_env.py` for Python commands** to load environment from `.env`:

```powershell
# ? CORRECT - Loads PYTHONPATH from .env
python scripts/run_with_env.py python -m pytest

# ? WRONG - Misses local development packages
python -m pytest
```

### Testing

```powershell
# Run all tests
python scripts/run_with_env.py python -m pytest --ff

# Run specific test
python scripts/run_with_env.py python -m pytest datalab/tests/features/signal/

# Coverage
python scripts/run_with_env.py python -m coverage run -m pytest datalab
python -m coverage html
```

### Linting and Formatting

**Prefer Ruff** (fast, modern):

```powershell
# Format code
python scripts/run_with_env.py python -m ruff format

# Lint with auto-fix
python scripts/run_with_env.py python -m ruff check --fix
```

**Pylint** (with extensive disables for code structure):

```powershell
python scripts/run_with_env.py python -m pylint datalab \
    --disable=duplicate-code,fixme,too-many-arguments,too-many-branches, \
    too-many-instance-attributes,too-many-lines,too-many-locals, \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DataLab-Platform/DataLab](https://github.com/DataLab-Platform/DataLab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
