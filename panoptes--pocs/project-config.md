---
trigger: always_on
description: manages the entire observation workflow through a state machine architecture.
---

# AI Agent Guidelines for POCS

This document provides guidelines for AI coding agents working with the PANOPTES Observatory Control System (POCS)
codebase. It is designed to be tool-agnostic and applicable to any AI assistant working on this project.

## Project Overview

POCS (PANOPTES Observatory Control System) is the main software driver for robotic astronomical observatories designed
to detect transiting exoplanets. The system controls telescope hardware, schedules observations, captures images, and
manages the entire observation workflow through a state machine architecture.

**Key Characteristics:**

- **Language:** Python 3.12+ (type hints expected)
- **Architecture:** State machine-based observatory control
- **Domain:** Astronomy, robotics, hardware control
- **Testing:** pytest with high coverage requirements
- **Package Manager:** uv (modern Python package manager)
- **Code Style:** Ruff for linting and formatting

## Essential Reading

Before making changes, review these documents:

1. **Architecture:** `docs/architecture-for-beginners.md` - Understand the layered architecture
2. **Contributing:** `CONTRIBUTING.md` - Development workflow and standards
3. **CLI Guide:** `docs/cli-guide.md` - Command-line interface reference
4. **Glossary:** `docs/glossary.md` - Domain-specific terminology
5. **Conceptual Overview:** `docs/conceptual-overview.md` - High-level system design

## Project Structure

```
POCS/
├── src/panoptes/pocs/          # Main source code
│   ├── core.py                 # POCS state machine (the brain)
│   ├── observatory.py          # Hardware coordinator
│   ├── scheduler/              # Observation scheduler
│   ├── camera/                 # Camera drivers
│   ├── mount/                  # Telescope mount drivers
│   ├── dome/                   # Dome control
│   ├── focuser/                # Focus control
│   └── utils/                  # Utilities and CLI
├── tests/                      # Test suite
├── conf_files/                 # Configuration files
├── docs/                       # Documentation
└── examples/                   # Example scripts
```

## Development Workflow

### 1. Understanding Changes

**Before making any changes:**

- Check if an issue exists for the change; reference it in commits/PRs
- Read relevant architecture documentation to understand affected components
- Review existing tests to understand expected behavior
- Check `pyproject.toml` for dependencies and project configuration

### 2. Code Standards

**Style and Formatting:**

- Use Ruff for linting and formatting (configured in `pyproject.toml`)
- Line length: 110 characters
- Quote style: double quotes
- Follow PEP 8 conventions

**Type Hints:**

- Required for all function signatures
- Use modern Python 3.12+ type syntax
- Import from `typing` when necessary

**Documentation:**

- Docstrings for all public classes and functions
- Use Google-style docstrings
- Include examples in docstrings when helpful

### 3. Testing Requirements

**All code changes must include tests:**

- Unit tests in `tests/` directory
- Test files named `test_*.py`
- Use pytest fixtures from `conftest.py`
- Maintain or improve code coverage
- Run tests locally before committing: `pytest`

**Testing markers available:**

```python
@pytest.mark.theskyx  # Tests requiring TheSkyX
@pytest.mark.with_camera  # Tests requiring camera hardware
@pytest.mark.without_camera  # Tests that should skip camera
@pytest.mark.plate_solve  # Tests requiring plate solving
```

### 4. Dependencies

**Adding Dependencies:**

- Add to `dependencies` in `pyproject.toml` for runtime requirements
- Add to `[dependency-groups]` for development/testing tools
- Use `uv add <package>` to install and update lockfile
- Pin security-sensitive packages (e.g., `certifi>=2024.2.2`)

**Optional Dependencies:**

- `focuser`: Matplotlib and focus-related tools
- `google`: Google Cloud integration
- `weather`: Weather station support
- `all`: All optional features

### 5. Making Changes

**File Editing Best Practices:**

1. Read entire files or large sections before editing
2. Preserve existing code style and patterns
3. Make minimal, focused changes
4. Validate changes by checking for errors after editing
5. Run relevant tests to confirm functionality

**Commit Messages:**

- Clear, descriptive commit messages
- Reference issue numbers when applicable
- Format: `Brief description (#issue-number)`

**Pre-commit Checklist:**

Before every commit, run lint and format to ensure no issues:

```bash
ruff check .
ruff format .
```

Both commands must pass cleanly before committing.

## Architecture Guidelines

### State Machine (POCS Core)

**Location:** `src/panoptes/pocs/core.py`

The POCS state machine orchestrates observations. Key states include:

- `sleeping` → `ready` → `scheduling` → `slewing` → `tracking` → `observing` → `parking`

**When modifying:**

- Understand state transitions (defined by `transitions` library)
- Respect the state flow logic
- Add appropriate state validation
- Update state documentation if adding new states

### Scheduler Component

**Location:** `src/panoptes/pocs/scheduler/`

The scheduler decides WHAT to observe (POCS decides WHEN).

**When modifying:**

- Understand constraints system

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [panoptes/POCS](https://github.com/panoptes/POCS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
