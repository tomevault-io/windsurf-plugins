---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Core Principles: Simplicity & Type Safety

KiLM (KiCad Library Manager) prioritizes **professional code standards** and **complete type safety**:

- **Type-first development**: Everything uses proper type hints - no `Any`, no dynamic typing
- **Professional standards**: No emojis in code, no hardcoded values, proper constants
- **CLI-focused**: Simple, reliable command-line interface for KiCad library management
- **Cross-platform**: Support for Windows, macOS, Linux KiCad installations

## Architecture Overview

KiLM is a **command-line tool for managing KiCad libraries** across projects and workstations:

**Core Functionality**:
- KiCad configuration detection across platforms
- Library management (symbol and footprint libraries)  
- Environment variable configuration
- Project template management
- Backup and restore of KiCad configurations

## Development Commands

### Python CLI Development
```bash
# Install Python package in development mode
pip install -e ".[dev]"

# Install pre-commit hooks (once implemented)
pre-commit install

# Test basic functionality
kilm status
kilm --help
```

### Testing & Quality
```bash
# Run tests with coverage
pytest --cov=kicad_lib_manager --cov-report=html

# Type checking, formatting, and linting
pyrefly    # Type check (required - no "any" types allowed)
ruff format .
ruff check .

# All quality checks
pre-commit run --all-files
```

### CI/CD and Releases
```bash
# Create a new release (automated via GitHub Actions)
# 1. Update version in kicad_lib_manager/__init__.py
# 2. Push version tag:
git tag v0.3.1
git push origin v0.3.1

# This triggers:
# - Automated testing and quality checks
# - PyPI publishing with trusted publishing
# - Draft GitHub release with auto-generated notes
# - Multi-platform compatibility verification
```

**Release Process:**
1. **Tag-based releases**: Push version tags to trigger automated releases
2. **Draft releases**: GitHub releases created as drafts for manual review
3. **Automatic PyPI**: Publishes to PyPI immediately via trusted publishing
4. **Auto-generated notes**: Release notes include commits, PRs, and install instructions

## Code Quality Standards

### Type Safety Requirements
- **No Any types**: All functions must have proper type hints
- **Pydantic models**: Use Pydantic for configuration validation where applicable
- **Type checking**: Must pass pyrefly type checking without errors

### Professional Code Standards
- **No emojis**: Keep code and output professional - avoid emojis in code, comments, or CLI output
- **No hardcoding**: Use constants, configuration files, or environment variables
- **Proper error handling**: Consistent error patterns with informative messages
- **Cross-platform paths**: Use pathlib.Path for all file operations
- **Context7**: Often use context 7 MCP when dealing with new code and packages

## CLI Architecture

### Command Structure
```
kilm                    # Main CLI entry point
├── init               # Initialize library
├── setup              # Configure KiCad to use libraries  
├── status             # Show current configuration
├── list               # List available libraries
├── pin/unpin          # Pin/unpin favorite libraries
├── add-3d             # Add 3D model libraries
├── config             # Configuration management
├── sync               # Update/sync library content (was 'update')
├── update             # Update KiLM itself (breaking change in 0.4.0)
├── add-hook           # Add project hooks
└── template           # Project template management
```

## BREAKING CHANGES in v0.4.0

### Command Restructuring
- **`kilm update`** now updates KiLM itself (self-update functionality)
- **`kilm sync`** updates library content (was `kilm update`)
- Added deprecation banner for transition period
- Full auto-update functionality with installation method detection

### New Features
- **Self-Update System**: Detects installation method (pip, pipx, conda, uv, homebrew)
- **PyPI Integration**: Checks for latest versions with proper caching
- **Update Preferences**: Configurable update checking and frequency
- **Professional UX**: Non-intrusive notifications with method-specific guidance

### Core Modules
- **CLI Layer** (`main.py`): Typer-based command interface with Rich output
- **Commands** (`commands/`): Individual command implementations
- **Library Manager** (`library_manager.py`): Core library management logic
- **Configuration** (`config.py`): KiCad configuration handling with update preferences
- **Auto-Update** (`auto_update.py`): Self-update functionality with installation detection
- **Utilities** (`utils/`): File operations, backups, metadata, templates

## Development Workflow - MANDATORY

### Task Documentation
- **ALWAYS create task file**: For ANY work request, immediately create `.claude/doc/tasks/[date]-[seq]-[task-name].md`
- **Update throughout**: Document progress, decisions, blockers in real-time
- **Include context**: Always pass current task file path to agents for context sharing

### Code Quality Workflow  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [barisgit/KiLM](https://github.com/barisgit/KiLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
