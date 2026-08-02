---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Dioptas is a GUI program for fast analysis of powder X-ray diffraction images. It provides calibration, masking, pattern integration, phase line overlays, and batch processing capabilities for synchrotron and laboratory X-ray diffraction data.

## Development Setup

This project uses the `uv` package manager (switched from poetry). The `develop` branch is the main development branch.

### Initial Setup

```bash
# Install uv
python -m pip install uv

# Install dependencies
uv sync

# Install test dependencies
uv sync --extra test
```

### Running the Application

```bash
# Run from source
uv run dioptas

# Alternative (without uv, if dependencies installed manually)
python run.py
```

## Common Commands

### Testing

```bash
# Run all tests
uv run pytest

# Run specific test suites
uv run pytest dioptas/tests/unit_tests/          # Model/utility tests
uv run pytest dioptas/tests/controller_tests/    # Controller tests
uv run pytest dioptas/tests/widget_tests/        # Widget tests
uv run pytest dioptas/tests/functional_tests/    # End-to-end tests

# Run a specific test file
uv run pytest dioptas/tests/unit_tests/test_DioptasModel.py

# Run a specific test function
uv run pytest dioptas/tests/unit_tests/test_DioptasModel.py::test_specific_function

# Note: pytest is configured with -v --tb=short in pytest.ini
```

### Building Executables

```bash
# PyInstaller spec file is provided: Dioptas.spec
pyinstaller Dioptas.spec

# Platform-specific scripts
create_executable.sh    # Linux/Mac
create_executable.bat   # Windows
```

## Architecture

Dioptas follows a strict **Model-View-Controller (MVC)** pattern:

### Directory Structure

```
dioptas/
├── model/              # Data layer - business logic, no Qt dependencies
├── controller/         # Controller layer - connects models to widgets
├── widgets/            # View layer - Qt GUI components
├── resources/          # Static resources (calibrants, icons, styles, data)
└── tests/              # Test suite organized by layer
```

### Model Layer (`dioptas/model/`)

The model layer handles all data and business logic:

- **DioptasModel**: Top-level model managing multiple configurations
- **Configuration**: Bundles related models (Image, Calibration, Mask, Pattern) into a working set
- Core models: `ImgModel`, `CalibrationModel`, `MaskModel`, `PatternModel`
- Feature models: `PhaseModel`, `OverlayModel`, `MapModel2`, `BatchModel`
- Utilities in `model/util/`: Signal system, corrections, JCPDS handling, calculations

**Key principles:**
- Models are pure Python (no Qt/GUI dependencies)
- Use custom `Signal` class (`model/util/signal.py`) for observer pattern with weak references
- Emit signals on data changes: `img_changed`, `pattern_changed`, `phase_added`, etc.
- Automatic re-integration: Calibration/mask changes trigger pattern regeneration

### View Layer (`dioptas/widgets/`)

Qt-based GUI components organized hierarchically:

- **MainWidget**: Top-level container with mode switching
- Mode widgets: `CalibrationWidget`, `MaskWidget`, `IntegrationWidget`, `MapWidget`
- **IntegrationWidget** structure:
  - `IntegrationControlWidget`: Control panels (tabs for Phase, Overlay, Background, etc.)
  - `IntegrationImgDisplayWidget`: Image/cake visualization
  - `IntegrationPatternWidget`: Pattern plot display
  - `IntegrationStatusWidget`: Status information

**Key principles:**
- Widgets contain only Qt/GUI code
- Use splitters for resizable layouts
- Parent widgets create "shortcuts" to deeply nested child widgets for convenience
- Named pattern: `{Feature}Widget`

### Controller Layer (`dioptas/controller/`)

Controllers orchestrate interaction between models and widgets:

- **MainController**: Application entry point, manages mode switching
- Mode controllers: `CalibrationController`, `MaskController`, `IntegrationController`, `MapController`
- **IntegrationController** creates specialized sub-controllers:
  - `PatternController`, `ImageController`, `OverlayController`, `PhaseController`
  - `BackgroundController`, `CorrectionController`, `OptionsController`, `BatchController`

**Key principles:**
- Controllers own all signal connections between models and widgets
- Named pattern: `{Feature}Controller`
- Implement `activate()` / `deactivate()` methods for mode switching
- Manage signal connections/disconnections to prevent memory leaks
- Subscribe to model signals and update views accordingly

### Signal-Based Communication

Data flow example (loading an image):
1. User clicks "Load Image" in widget
2. `ImageController` handles click, shows file dialog
3. Controller calls `img_model.load(filename)`
4. `ImgModel` loads file and emits `img_changed` signal
5. Multiple listeners respond:
   - `CalibrationModel` auto-integrates to generate pattern
   - `PatternModel` emits `pattern_changed`
   - Controllers update widget displays

### Configuration System

- `Configuration` class bundles Image, Calibration, Mask, and Pattern models
- `DioptasModel` manages multiple configurations simultaneously
- Enables side-by-side comparison of different analysis setups

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dioptas/Dioptas](https://github.com/Dioptas/Dioptas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
