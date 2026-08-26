---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Rejoice Slim v2** - Local-first voice transcription tool. Terminal-only, privacy-first, zero data loss.

- **Name Meaning:** Rejoice = {record, jot, voice}, Slim = {super lightweight, no UI, no cloud, no crazy}
- **Core Philosophy:** Data integrity above all, simplicity over features, local-first/local-only, transparency over magic, boring reliability over clever features
- **Status:** In active development (Phase 1 complete, Phase 2 in progress)
- **License:** MIT

## Development Commands

### Environment Setup
```bash
# Activate virtual environment
source venv/bin/activate

# Install dependencies (development)
pip install -e ".[dev]"

# Install pre-commit hooks
pre-commit install
```

### Testing Commands
```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=src/rejoice --cov-report=html --cov-report=term

# Run specific test types
pytest tests/unit           # Unit tests only
pytest tests/integration    # Integration tests only
pytest tests/e2e            # End-to-end tests only

# Run single test
pytest tests/unit/test_config.py::test_load_default_config -v

# Run tests in watch mode (if pytest-watch installed)
ptw tests/unit
```

### Code Quality
```bash
# Format code
black src tests

# Lint
flake8 src tests

# Type check
mypy src

# Run all pre-commit checks
pre-commit run --all-files
```

### Running the CLI
```bash
# Using the installed command (after installation)
rec --help
rec --version
rec config show

# During development (from project root)
python -m rejoice --help
python -m rejoice config show
```

### Installation Testing
```bash
# Test installation script
bash scripts/install.sh

# Test uninstallation script
bash scripts/uninstall.sh

# Verify setup
bash scripts/verify_setup.sh
```

## Architecture Overview

### Technology Stack
- **faster-whisper** (4x faster than openai-whisper) - Transcription engine with VAD support
- **sounddevice + portaudio** - Concurrent audio recording
- **ollama** - Local AI enhancement (summaries, tags, titles)
- **click** - CLI framework
- **rich** - Terminal UI (progress indicators, formatting)
- **pyyaml** - Configuration management
- **pytest** - Testing framework

### Key Design Patterns

#### Zero Data Loss Architecture
**Critical:** Files are created immediately when recording starts, not at the end. All transcript operations use atomic writes to prevent corruption.

```python
# CORRECT: Create file first
filepath, tid = create_transcript()  # File exists NOW
start_audio_capture()                # Then start recording

# WRONG: Create file at end
start_audio_capture()                # Recording starts
save_transcript()                    # ← Crash here = total data loss
```

#### Configuration Hierarchy
Configuration loads in this order (later overrides earlier):
1. Hardcoded defaults (in `core/config.py`)
2. User config file (`~/.config/rejoice/config.yaml`)
3. Environment variables (`.env` file or shell)
4. Command-line flags

#### Test-Driven Development (TDD)
**Required workflow for all new features:**
1. Write failing test first
2. Implement minimal code to pass
3. Refactor while keeping tests green
4. Coverage target: 90%+

### Directory Structure
```
src/rejoice/
├── __init__.py           # Version, exports
├── __main__.py           # CLI entry point
├── cli/                  # CLI commands
│   ├── commands.py       # Main command definitions
│   └── config_commands.py # Config subcommands
├── core/                 # Core functionality
│   ├── config.py         # Configuration system
│   └── logging.py        # Logging setup
├── audio/                # Audio recording (Phase 2)
├── transcription/        # Whisper integration (Phase 3)
├── transcript/           # MD file management
├── ai/                   # Ollama integration (Phase 5)
├── utils/                # Utilities
└── exceptions.py         # Custom exceptions

tests/
├── unit/                 # Fast, isolated tests
├── integration/          # Multi-component tests
├── e2e/                  # Full system tests
├── fixtures/             # Test data
└── conftest.py           # Pytest configuration
```

## Critical Implementation Rules

### 1. Never Lose Data
- Create transcript files immediately when recording starts
- Use atomic writes for all file operations (write to temp, then rename)
- Append transcription segments incrementally, not all at once
- Files must survive crashes, interruptions, power loss

### 2. Respect the "Slim" Mandate
**Never add these:**
- GUI or web interface (terminal only)
- Cloud services or remote APIs
- Database (flat markdown files only)
- Complex plugin systems

### 3. Testing Requirements
- Write tests BEFORE implementation (TDD)
- Unit test coverage: 90%+ required
- Integration tests for all user flows
- E2E tests for critical paths (recording, transcription, AI)
- All new features must have tests

### 4. Configuration Management
- All settings in `~/.config/rejoice/config.yaml`
- Support environment variable overrides (prefix: `REJOICE_`)
- Validate configuration on load
- Provide clear error messages for invalid config

### 5. Error Handling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lenniott/rejoice-slim-v2](https://github.com/Lenniott/rejoice-slim-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
