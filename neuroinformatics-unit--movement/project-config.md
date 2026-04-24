---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`movement` is a Python toolbox for analysing animal body movements across space and time. It provides a standardised, modular interface for processing motion tracking data from tools like DeepLabCut, SLEAP, LightningPose, Anipose, and VIA-tracks. The package handles data loading, cleaning, transformation, kinematic analysis, and visualization.

## Development Commands

### Environment Setup
```bash
# Create virtual environment with uv
uv venv --python 3.13
source .venv/bin/activate  # On macOS and Linux
.venv\Scripts\activate     # On Windows PowerShell

# Install in editable mode with dev dependencies
uv pip install -e ".[dev]"

# Install pre-commit hooks
pre-commit install
```

### Testing
```bash
# Run all tests with coverage
pytest

# Run specific test file
pytest tests/test_unit/test_io/test_load_poses.py

# Run specific test function
pytest tests/test_unit/test_io/test_load_poses.py::test_from_dlc_file

# Run tests matching pattern
pytest -k "test_load"

# Run with verbose output
pytest -v

# Run without coverage (faster)
pytest --no-cov
```

### Linting and Formatting
```bash
# Run pre-commit hooks on staged files
pre-commit run

# Run pre-commit hooks on all files
pre-commit run -a

# Auto-fix with ruff
ruff check --fix

# Type checking with mypy
mypy movement
```

### Documentation
```bash
# Install documentation dependencies
pip install -e ".[docs]"

# Build documentation (from docs/ directory)
cd docs
make html

# Clean and rebuild documentation
make clean html

# Check external links
make linkcheck

# View built documentation
open build/html/index.html  # macOS
```

## High-Level Architecture

### Core Data Structure: xarray Datasets

The entire package is built around **xarray Datasets** as the central data structure. Understanding this is crucial for working with the codebase.

**Poses Dataset:**
- Dimensions: `(time, space, keypoints, individuals)`
- Variables:
  - `position`: shape `(n_frames, n_space, n_keypoints, n_individuals)`
  - `confidence`: shape `(n_frames, n_keypoints, n_individuals)`
- Coordinates:
  - `time`: seconds (if fps provided) or frame numbers
  - `space`: `["x", "y"]` or `["x", "y", "z"]`
  - `keypoints`: list of keypoint names
  - `individuals`: list of individual names
- Attributes: `source_software`, `fps`, `time_unit`, `ds_type="poses"`, `log`

**Bboxes Dataset:**
- Dimensions: `(time, space, individuals)`
- Variables:
  - `position`: centroid positions
  - `shape`: bounding box dimensions
  - `confidence`: detection confidence

### Module Organization

**`io/`** - Data loading and saving
- `load.py`: Unified entry points `load_dataset()` and `load_multiview_dataset()` with a `@register_loader` decorator pattern for format-specific loaders
- `load_poses.py`: Format-specific pose loaders (`from_dlc_file()`, `from_sleap_file()`, `from_lp_file()`, `from_anipose_file()`). Note: `from_file()` and `from_multiview_files()` are deprecated in favour of `load_dataset()` / `load_multiview_dataset()`
- `load_bboxes.py`: Format-specific bbox loaders, including VIA-tracks support. Note: `from_file()` is deprecated in favour of `load_dataset()`
- `save_poses.py`, `save_bboxes.py`: Export to various formats
- `nwb.py`: NWB (Neurodata Without Borders) format support

**`validators/`** - Multi-layer validation system
- `files.py`: File format validation using attrs classes (`ValidFile`, `ValidHDF5`, `ValidDeepLabCutCSV`, etc.)
- `datasets.py`: Dataset validation (`ValidPosesDataset`, `ValidBboxesDataset`) with automatic defaults and shape checking
- `arrays.py`: Array validation (`validate_dims_coords()`)

**`filtering.py`** - Signal processing
- `filter_by_confidence()`: Mask low-confidence points
- `interpolate_over_time()`: Fill NaN gaps (linear, nearest, cubic)
- `savgol_filter()`, `rolling_filter()`: Smooth trajectories

**`transforms.py`** - Spatial transformations
- `scale()`: Convert pixel to physical units
- `compute_homography_transform()`: Perspective correction

**`kinematics/`** - Motion analysis
- `kinematics.py`: Core kinematics (velocity, acceleration, displacement)
- `distances.py`: Pairwise distances between keypoints
- `orientation.py`: Forward vectors, head direction
- `kinetic_energy.py`: Energy computations

**`roi/`** - Region of Interest analysis
- `base.py`: Abstract base class for ROIs
- `polygon.py`: Polygon regions (occupancy, crossing events)
- `line.py`: Line regions (crossing detection)
- `conditions.py`: Boolean conditions on ROIs

**`napari/`** - GUI plugin
- `meta_widget.py`: `MovementMetaWidget` - collapsible container wrapping all napari widgets
- `loader_widgets.py`: File loading widgets
- `convert.py`: xarray → napari layer conversion
- `layer_styles.py`: Visual styling for layers
- `regions_widget.py`: Interactive region drawing and analysis

**`plots/`** - Visualization
- `trajectory.py`: Plot trajectories over time
- `occupancy.py`: Heatmaps of spatial occupancy

**`utils/`** - Cross-cutting utilities
- `logging.py`: `MovementLogger` wrapper around loguru with `log_to_attrs()` decorator for operation provenance

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/neuroinformatics-unit) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
