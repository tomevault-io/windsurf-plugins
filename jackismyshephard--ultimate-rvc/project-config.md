---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ultimate RVC is an AI voice cloning and conversion tool that extends AiCoverGen with improved features. It provides a comprehensive suite for:

- **Voice conversion**: Converting vocals using RVC (Retrieval-based Voice Conversion) models
- **Song cover generation**: Creating AI covers from YouTube videos or uploaded audio
- **Text-to-speech**: Generating speech from text using voice models  
- **Voice model training**: Training custom RVC models from audio datasets
- **Audio processing**: Advanced vocal extraction, noise reduction, pitch correction

The project supports both web UI (Gradio) and CLI interfaces, with GPU acceleration via CUDA/ROCm.

## Development Commands

### Setup and Environment

- `./urvc install` - Install all dependencies and set up environment (Linux/WSL)
- `./urvc.ps1 install` - Install dependencies (Windows PowerShell)
- `./urvc update` - Update to latest version
- `./urvc uninstall` - Clean uninstall of dependencies and data

### Running the Application

- `./urvc run` - Start web UI at <http://127.0.0.1:7860>
- `./urvc cli [subcommand]` - Access CLI interface
- `./urvc-web` - Start web UI (when installed as PyPI package)
- `urvc` - CLI interface (when installed as PyPI package)

### Development Tools

- `./urvc dev` - Start in development mode with hot reloading
- `./urvc docs [module] [output_dir]` - Generate documentation using Typer
- `./urvc uv [command]` - Run arbitrary uv commands
  - Code linting: `uv run ruff check` (configured in pyproject.toml)
  - Type checking: `uv run pyright` (Pyright configuration in pyproject.toml)
  - Code formatting: `uv run ruff format` (Black-compatible formatting)

### Testing

The project uses pytest for comprehensive testing with unit, integration, and end-to-end test coverage.

**Running Tests:**

```bash
# Run all tests
./urvc uv run pytest

# Run specific test file
./urvc uv run pytest tests/unit/core/test_common.py

# Run with verbose output
./urvc uv run pytest -v

# Run specific test class or method
./urvc uv run pytest tests/unit/core/test_common.py::TestDisplayProgress
./urvc uv run pytest tests/unit/core/test_common.py::TestDisplayProgress::test_display_progress_with_message_only

# Run tests matching pattern
./urvc uv run pytest -k "test_display_progress"

# Run with coverage report
./urvc uv run pytest --cov

# Run and stop on first failure
./urvc uv run pytest -x

# Run specific test categories
./urvc uv run pytest -m "not slow"     # Exclude slow tests
./urvc uv run pytest -m "network"     # Only network tests
./urvc uv run pytest -m "gpu"         # Only GPU tests
```

**Test Structure:**

- `tests/unit/` - Unit tests (isolated function testing)
- `tests/integration/` - Integration tests (module interactions)  
- `tests/e2e/` - End-to-end tests (full workflows)

**Test Requirements:**

- All tests must pass pre-commit hooks (linting, type checking)
- Minimum 90% test coverage required
- Use realistic data for integration/E2E tests, synthetic for unit tests

**Critical Testing Principles:**

- **NEVER change application code without EXPLICIT permission from user**
- **Always test ALL cases**: positive, negative, and edge cases for each function
- **Look for bugs in functions being tested** - most are correct, but some may have issues
- **Report any suspected bugs** but do not fix without permission
- **Test comprehensively** - every function parameter, return value, and exception path

## Architecture Overview
  
### Core Structure

```console
src/ultimate_rvc/
├── cli/           # Command-line interface using Typer
├── core/          # Core business logic
│   ├── generate/  # Audio generation pipelines
│   ├── manage/    # Model and data management  
│   └── train/     # Model training workflows
├── rvc/           # RVC model implementation
│   ├── configs/   # Sample rate configurations (32k/40k/48k)
│   ├── infer/     # Inference pipeline and models
│   ├── lib/       # Core algorithms, predictors, utilities
│   └── train/     # Training data processing and model training
├── web/           # Gradio web interface
│   ├── config/    # UI configuration management
│   └── tabs/      # Modular UI tabs (generate, manage, train)
└── stubs/         # Type stubs for external libraries
```

### Key Components

**Audio Processing Pipeline** (`core/generate/`):

- Vocal extraction using audio-separator with UVR models
- Pitch extraction with FCPE/RMVPE methods
- Voice conversion using RVC models
- Audio effects and post-processing

**Model Management** (`core/manage/`):

- Voice model downloading from URLs/AI Hub Discord
- Model upload and validation
- Pretrained model management
- Custom configuration persistence

**Training System** (`rvc/train/`):

- Dataset preparation and audio slicing
- Feature extraction with different embedders (ContentVec, HuBERT variants)
- RVC model training with configurable parameters
- Index creation for voice similarity matching

**Web Interface** (`web/`):

- Modular tab-based UI using Gradio 5
- Real-time configuration updates
- Audio preview and download capabilities

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JackismyShephard/ultimate-rvc](https://github.com/JackismyShephard/ultimate-rvc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
