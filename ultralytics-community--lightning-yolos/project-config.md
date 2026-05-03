---
trigger: always_on
description: Lightning-YOLOs is a YOLO-OBB (Oriented Bounding Box) training framework built with PyTorch Lightning. The project provides a modular, production-ready implementation for training YOLO models with oriented bounding box detection capabilities.
---

# GitHub Copilot Instructions for Lightning-YOLOs

## Project Overview

Lightning-YOLOs is a YOLO-OBB (Oriented Bounding Box) training framework built with PyTorch Lightning. The project provides a modular, production-ready implementation for training YOLO models with oriented bounding box detection capabilities.

**Key Technologies:**

- PyTorch Lightning (>=2.0.0, \<2.6.0)
- Ultralytics YOLO (licensed under AGPL-3.0)
- TorchMetrics with detection extras (required for mAP metrics)
- jsonargparse for CLI configuration

**Project Structure:**

The main package is located in `src/lit_yolo/` with a modular architecture:

- Core modules for data loading, model definitions, and training orchestration
- CLI entry point via `__main__.py`
- Package exports through `__init__.py`

Check the actual directory for the current module organization as it may evolve with subpackages.

> **Note:** When making structural changes to the project (adding/removing modules, reorganizing directories, changing workflows), remember to update all relevant documentation including this file, README.md, src/README.md, and AGENTS.md.

## Python Version & Dependencies

- **Python**: >=3.10 (supports 3.10, 3.11, 3.12)
- **Main Dependencies**: pytorch-lightning, ultralytics, jsonargparse, torchmetrics, opencv-python
- **Install**: `pip install -e .[dev]` for development with all dependencies

## Coding Standards

### Style Guidelines

- **Formatter**: Ruff (configured in `pyproject.toml`)
- **Line Length**: 120 characters
- **Quote Style**: Double quotes
- **Import Order**: Enforced by Ruff (E, F, W, I, N, UP rules)
- **Type Hints**: Use type hints where possible for function signatures
- **Docstrings**: Use for all public functions and classes

### Code Quality

- Follow PEP 8 style guidelines
- Use descriptive variable names
- Keep functions focused and modular
- Add comments only when necessary to explain complex logic
- Use logging instead of print statements (configured format: `%(asctime)s | %(levelname)-8s | %(name)s | %(message)s`)

### Pre-commit Hooks

The project uses pre-commit hooks that run automatically on commit:

- `ruff` for linting and formatting
- `codespell` for spell checking
- `docformatter` for docstring formatting
- `mdformat` for markdown formatting
- File hygiene checks (trailing whitespace, end-of-file-fixer, etc.)

**Run manually:**

```bash
pre-commit run --all-files
```

## Testing

### Test Structure

Tests are organized in the `tests/` directory with separate subdirectories for unit tests and integration tests. Check the directory structure for the current organization.

### Running Tests

```bash
# Run all tests
pytest .

# Run specific test directory
pytest tests/unittests/
pytest tests/integrations/

# Run with coverage
pytest tests/ --cov=lit_yolo

# Run doctests in source
pytest src/
```

### Test Configuration

- Configured in `pyproject.toml` under `[tool.pytest.ini_options]`
- Doctests enabled with `--doctest-modules`
- Verbose output with color enabled
- Test files: `test_*.py`
- Test functions: `test_*`
- Test classes: `Test*`

### Writing Tests (Required)

All code changes must include appropriate tests:

- Write unit tests for new features (required)
- Maintain or improve test coverage
- Use descriptive test names that explain what is being tested
- Follow existing test patterns in the repository
- Test both success and error conditions

## Development Workflow

### Setting Up Development Environment

```bash
# Clone the repository
git clone https://github.com/Borda/lightning-YOLOs.git
cd lightning-YOLOs

# Install with development dependencies
pip install -e .[dev]

# Install pre-commit hooks
pre-commit install

# Verify installation
pytest .
```

### Making Changes

1. Create a feature branch from `main`
2. Make your changes following the coding standards
3. Write tests for new functionality
4. Run linting: `pre-commit run --all-files`
5. Run tests: `pytest .`
6. Commit changes (pre-commit hooks run automatically)
7. Submit a pull request

### CLI Usage Examples

```bash
# Train with default settings
lit-yolo train --data /path/to/dataset --model yolo11n-obb.pt

# Train with custom parameters
lit-yolo train --data /path/to/dataset --model yolo11n-obb.pt --epochs 50 --batch_size 16

# Train with config file
lit-yolo train --config config.yaml

# Run as module (without installation)
python -m lit_yolo train --data /path/to/dataset --model yolo11n-obb.pt
```

## Architecture & Design Patterns

### Lightning Module Structure

The `LitYOLOOBB` class in `models.py` follows standard PyTorch Lightning patterns with YOLO-OBB specific implementations for training and validation with mAP metrics.

### Data Module

The `OBBDataModule` in `data.py` handles:

- Auto-detection of number of classes from dataset
- Batch loading with proper collation
- Data preprocessing and augmentation via YOLO

### Training Function

The `train()` function in `training.py` orchestrates:

- Model and data module initialization
- Trainer configuration with callbacks
- Training execution
- Model export to ONNX/TorchScript


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ultralytics-community/lightning-YOLOs](https://github.com/Ultralytics-community/lightning-YOLOs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
