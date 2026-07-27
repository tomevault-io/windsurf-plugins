---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`ripple_detection` is a Python package for detecting sharp-wave ripple events (150-250 Hz) from local field potentials (LFPs) in neuroscience research. It implements detection algorithms from Karlsson et al. 2009 and Kay et al. 2016, along with other variants.

## Development Commands

### Setup

```bash
# Install from source (development mode with dev dependencies)
pip install -e .[dev,examples]

# Or create conda environment with all dependencies
conda env create -f environment.yml
conda activate ripple_detection
pip install -e .[dev,examples]

# Minimal install (runtime dependencies only)
pip install -e .
```

### Testing

```bash
# Run all tests with coverage (93% coverage achieved!)
pytest --cov=ripple_detection tests/

# Run specific test module
pytest tests/test_core.py          # Core signal processing tests
pytest tests/test_detectors.py     # Detector integration tests
pytest tests/test_simulate.py      # Simulation module tests

# Run specific test class or function
pytest tests/test_core.py::TestGetEnvelope
pytest tests/test_detectors.py::TestKayRippleDetector::test_single_channel_with_ripples

# Generate HTML coverage report
pytest --cov=ripple_detection --cov-report=html tests/
open htmlcov/index.html

# Test notebooks (as done in CI)
jupyter nbconvert --to notebook --ExecutePreprocessor.kernel_name=python3 --execute examples/detection_examples.ipynb
jupyter nbconvert --to notebook --ExecutePreprocessor.kernel_name=python3 --execute examples/test_individual_algorithm_components.ipynb
jupyter nbconvert --to notebook --ExecutePreprocessor.kernel_name=python3 --execute examples/ripple_detection_tutorial.ipynb
```

### Code Quality

```bash
# Format code with black
black ripple_detection/ tests/

# Check formatting without modifying files
black --check ripple_detection/ tests/

# Lint code with ruff (fast, modern linter - replaces flake8)
ruff check ripple_detection/ tests/

# Auto-fix ruff issues where possible
ruff check --fix ripple_detection/ tests/

# Type check with mypy
mypy ripple_detection/
```

### Building

```bash
# Build package using modern build tools (recommended)
python -m build

# Build with hatch (if installed)
hatch build
```

### Release Process

When preparing a new release:

```bash
# 1. Run all tests to ensure everything passes
pytest --cov=ripple_detection tests/

# 2. Run code quality checks
black --check ripple_detection/ tests/
ruff check ripple_detection/ tests/
mypy ripple_detection/

# 3. Update CHANGELOG.md
# - Add new version section with date: ## [X.Y.Z] - YYYY-MM-DD
# - Document all changes under appropriate headers:
#   - Added (new features)
#   - Changed (changes to existing functionality)
#   - Deprecated (soon-to-be removed features)
#   - Removed (removed features)
#   - Fixed (bug fixes)
#   - Security (security fixes)
# - List closed issues: "Closes #N"
# - Update comparison links at bottom of file

# 4. Commit the changelog
git add CHANGELOG.md
git commit -m "Update CHANGELOG for vX.Y.Z release"
git push origin master

# 5. Create and push annotated git tag
git tag -a vX.Y.Z -m "Release vX.Y.Z

## New Features
- Feature description

## Improvements
- Improvement description

Closes #N"

git push origin vX.Y.Z

# The tag push triggers the automated GitHub Actions release workflow:
# - Runs tests on Python 3.10, 3.11, 3.12, 3.13
# - Builds source distribution and wheels
# - Publishes to PyPI
# - Creates GitHub release with auto-generated notes
```

**Important Notes:**
- Always update CHANGELOG.md BEFORE creating the tag
- The tag must be an annotated tag (use `-a` flag) with a meaningful message
- Version follows semantic versioning (MAJOR.MINOR.PATCH)
- The version in `ripple_detection/_version.py` is auto-generated from the git tag by hatch-vcs
- Monitor the release workflow at: https://github.com/Eden-Kramer-Lab/ripple_detection/actions

## Architecture

### Core Module Structure

The package is organized into three main modules:

1. **[ripple_detection/core.py](ripple_detection/core.py)** - Low-level signal processing utilities
   - Bandpass filtering for ripple band (150-250 Hz)
   - Envelope extraction via Hilbert transform
   - Gaussian smoothing
   - Threshold detection and segment extraction
   - Movement exclusion based on speed
   - Utility functions for time series segmentation

2. **[ripple_detection/detectors.py](ripple_detection/detectors.py)** - High-level detection algorithms
   - `Kay_ripple_detector` - Multi-channel consensus approach (Kay et al. 2016)
   - `Karlsson_ripple_detector` - Per-channel detection with merging (Karlsson et al. 2009)
   - `Roumis_ripple_detector` - Variant detection method
   - `multiunit_HSE_detector` - Multiunit High Synchrony Event detector
   - All detectors return pandas DataFrames with event statistics

3. **[ripple_detection/simulate.py](ripple_detection/simulate.py)** - Synthetic data generation
   - Simulate LFPs with embedded ripples
   - Multiple noise types (white, pink, brown)
   - Used for testing and validation

### Detection Pipeline Architecture

All ripple detectors follow a common pipeline:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Eden-Kramer-Lab/ripple_detection](https://github.com/Eden-Kramer-Lab/ripple_detection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
