---
trigger: always_on
description: NVIDIA NeMo Curator is a generic, pluggable framework for building distributed data processing pipelines using Ray. It provides a unified API for running the same pipeline logic across different Ray orchestration backends.
---

# GitHub Copilot Instructions for NVIDIA NeMo Curator

## Overview

NVIDIA NeMo Curator is a generic, pluggable framework for building distributed data processing pipelines using Ray. It provides a unified API for running the same pipeline logic across different Ray orchestration backends.

### 🎯 Core Concept
Enable users to define a data processing pipeline once and execute it using different Ray backends in a multi-node setting (Xenna, Ray Actors, Ray Data) without changing the pipeline logic.

This scalable data preprocessing tool focuses on data curation pipelines for text, audio, video, and image modalities with support for both CPU and GPU processing.

## Development Environment

### Python Environment
- **Python Version**: 3.13 (recommended and tested), supports 3.11-3.13
- **Package Manager**: [uv](https://docs.astral.sh/uv/) for fast, reliable dependency management
- **Virtual Environment**: Use uv's built-in virtual environment management

### CUDA Environment (Optional)
- **CUDA Version**: 12.x (12.0+) for GPU acceleration
- **GPU Requirements**: NVIDIA GPU with Volta™ architecture or higher (compute capability 7.0+)
- **GPU Libraries**: RAPIDS (cuDF, cuML), PyTorch with CUDA 12 support, CuPy
- **Note**: GPU features are optional and CPU fallbacks are provided

## Key Technologies and Frameworks

### Core Dependencies
- **PyTorch**: Deep learning framework with CUDA support
- **Ray**: Distributed computing framework for data processing
- **Pandas/CuDF**: Data manipulation (CPU/GPU respectively)
- **Transformers**: Hugging Face transformers library
- **Loguru**: Structured logging

### Modality-Specific Libraries
- **Text Processing**: PyTorch, BeautifulSoup, fasttext, sentencepiece, trafilatura
- **Audio Processing**: NeMo Toolkit ASR components
- **Video Processing**: OpenCV, PyAV, CvCuda, PyNvVideoCodec
- **Image Processing**: NVIDIA DALI for optimized data loading

### Testing Framework
- **pytest**: Primary testing framework
- **pytest-asyncio**: Async testing support
- **pytest-coverage**: Code coverage measurement
- **GPU Testing**: Use `@pytest.mark.gpu` for GPU-dependent tests

## Setup Instructions

### Basic Setup
```bash
# Install uv package manager
pip3 install uv

# Clone and setup development environment
git clone <repository-url>
cd Curator
uv sync

# For GPU development (requires CUDA 12.x)
uv sync --extra deduplication_cuda12x
```

### Optional Feature Groups
```bash
# Text processing capabilities
uv sync --extra text

# Video processing with GPU acceleration
uv sync --extra video --extra video_cuda

# All features (includes CUDA dependencies)
uv sync --extra all
```

## Coding Standards and Patterns

### Code Quality
- **Linting**: Ruff with comprehensive rule set (see pyproject.toml)
- **Line Length**: 119 characters maximum
- **Type Hints**: Use comprehensive type annotations
- **Imports**: Follow import sorting conventions

### Module Structure
- **Stages**: Processing stages organized by modality (`text/`, `video/`, `audio/`, `image/`)
- **Utils**: Shared utilities for common operations
- **Datasets**: Data loading and manipulation classes
- **Modules**: Core processing algorithms

### Error Handling Patterns
```python
# Standard error handling for missing dependencies
try:
    import required_library
except ImportError as e:
    logger.error(f"Required dependency not found: {e}")
    raise ImportError("Please install the required dependencies")
```

### Configuration Patterns
- Use YAML configuration files for processing pipelines
- Support hierarchical configuration (CLI args > env vars > config files > defaults)
- Follow the configuration structure in `docs/admin/config/`

## Testing Guidelines

### Test Organization
- **Unit Tests**: Test individual functions and classes
- **Integration Tests**: Test complete processing pipelines
- **GPU Tests**: Mark with `@pytest.mark.gpu` decorator
- **Mock External Dependencies**: Use pytest mocks for external services

### Test Environment Setup
```python
# Example GPU test structure
@pytest.mark.gpu
def test_gpu_processing():
    import cudf  # Only import in GPU tests
    df = cudf.DataFrame({"col1": [1, 2, 3], "col2": [4, 5, 6]})
    assert len(df) == 3
```

### Test Configuration
- Tests use a unified Ray cluster configuration via `conftest.py`
- GPU availability is automatically detected using multiple methods (pynvml, nvidia-ml-py)
- Tests gracefully handle missing GPU dependencies

### Running Tests
```bash
# Run all tests (CPU only by default)
uv run pytest

# Run GPU tests (requires CUDA environment)
uv run pytest -m gpu

# Run specific test categories
uv run pytest -m "not gpu"  # CPU tests only

# Run tests for specific modules
uv run pytest tests/stages/text/
uv run pytest tests/stages/image/
```

## Build and Development

### Development Workflow
```bash
# Install development dependencies
uv sync

# Run linting and formatting
uv run ruff check .
uv run ruff format .

# Run tests
uv run pytest

# Run specific test modules
uv run pytest tests/utils/test_nvcodec_utils.py

# Local docs dev server (Fern; see fern/README.md)
make docs


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA-NeMo/Curator](https://github.com/NVIDIA-NeMo/Curator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
