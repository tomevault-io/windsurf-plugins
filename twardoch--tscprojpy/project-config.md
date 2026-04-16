---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 1. Project Overview

`tscprojpy` is a Python package that parses and manipulates Camtasia `.tscproj` project files (JSON format). The main functionality is scaling the canvas and all object positions/sizes by a given factor.

## 2. Key Commands

### 2.1. Development Setup

```bash
# Install dependencies with uv (preferred)
uv sync

# Or with standard pip
pip install -e ".[dev]"
```

### 2.2. Running Tests

```bash
# Run all tests with pytest
python -m pytest

# Run tests with verbose output
python -m pytest -v

# Run tests with coverage
python -m pytest --cov=tscprojpy
```

### 2.3. Code Quality

```bash
# Check code with ruff
python -m ruff check src/

# Format code with ruff
python -m ruff format src/

# Fix auto-fixable issues
python -m ruff check --fix src/
```

### 2.4. Running the CLI

```bash
# Install the package in development mode first
pip install -e .

# Then use the CLI
tscprojpy xyscale --input example/2507vlteas2a.tscproj --output scaled.tscproj --scale 150.0

# Or run directly with python
python -m tscprojpy.cli xyscale --input example/2507vlteas2a.tscproj --scale 150.0
```

## 3. Project Structure

The project follows a standard Python package layout:

- `src/tscprojpy/` - Main package source code
  - `__init__.py` - Package initialization with version handling
  - `cli.py` - Fire-based CLI implementation (currently has placeholder hello/version commands)
- `tests/` - Test files
- `example/` - Contains `2507vlteas2a.tscproj` - a real Camtasia 2025 project file example
- `pyproject.toml` - Project configuration using hatchling build system with hatch-vcs for versioning

## 4. Architecture Notes

1. **CLI Framework**: Uses Fire for CLI and Rich for terminal output
2. **Version Management**: Uses hatch-vcs to generate version from git tags
3. **Testing**: pytest with coverage reporting configured
4. **Linting/Formatting**: Ruff configured for Python 3.12+ with standard rules
5. **Camtasia Project Format**: `.tscproj` files are JSON with a specific structure containing:
   - Canvas dimensions (width, height)
   - Object positions and sizes in sourceBin, library items, and timeline tracks
   - Media references with rect coordinates

## 5. Implementation Status

Currently, only the project skeleton exists. The `xyscale` functionality needs to be implemented in `cli.py` to:

1. Load the JSON from the input .tscproj file
2. Scale the canvas dimensions
3. Recursively find and scale all position/size values (rect arrays, x/y coordinates)
4. Save the modified JSON to the output file

# Software Development Rules

## 6. Pre-Work Preparation

### 6.1. Before Starting Any Work

- **ALWAYS** read `WORK.md` in the main project folder for work progress
- Read `README.md` to understand the project
- STEP BACK and THINK HEAVILY STEP BY STEP about the task
- Consider alternatives and carefully choose the best option
- Check for existing solutions in the codebase before starting

### 6.2. Project Documentation to Maintain

- `README.md` - purpose and functionality
- `CHANGELOG.md` - past change release notes (accumulative)
- `PLAN.md` - detailed future goals, clear plan that discusses specifics
- `TODO.md` - flat simplified itemized `- [ ]`-prefixed representation of `PLAN.md`
- `WORK.md` - work progress updates

## 7. General Coding Principles

### 7.1. Core Development Approach

- Iterate gradually, avoiding major changes
- Focus on minimal viable increments and ship early
- Minimize confirmations and checks
- Preserve existing code/structure unless necessary
- Check often the coherence of the code you're writing with the rest of the code
- Analyze code line-by-line

### 7.2. Code Quality Standards

- Use constants over magic numbers
- Write explanatory docstrings/comments that explain what and WHY
- Explain where and how the code is used/referred to elsewhere
- Handle failures gracefully with retries, fallbacks, user guidance
- Address edge cases, validate assumptions, catch errors early
- Let the computer do the work, minimize user decisions
- Reduce cognitive load, beautify code
- Modularize repeated logic into concise, single-purpose functions
- Favor flat over nested structures

## 8. Tool Usage (When Available)

### 8.1. Additional Tools

- If we need a new Python project, run `curl -LsSf https://astral.sh/uv/install.sh | sh; uv venv --python 3.12; uv init; uv add fire rich; uv sync`
- Use `tree` CLI app if available to verify file locations
- Check existing code with `.venv` folder to scan and consult dependency source code
- Run `DIR="."; uvx codetoprompt --compress --output "$DIR/llms.txt" --respect-gitignore --cxml --exclude "*.svg,.specstory,*.md,*.txt,ref,testdata,*.lock,*.svg" "$DIR"` to get a condensed snapshot of the codebase into `llms.txt`

## 9. File Management

### 9.1. File Path Tracking

- **MANDATORY**: In every source file, maintain a `this_file` record showing the path relative to project root
- Place `this_file` record near the top:
- As a comment after shebangs in code files
- In YAML frontmatter for Markdown files
- Update paths when moving files
- Omit leading `./`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/twardoch) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
