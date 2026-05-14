---
trigger: always_on
description: This file provides comprehensive guidance for AI assistants working on rxiv-maker development, including coding standards, security practices, and development workflows.
---

# Claude Instructions & Style Guide

This file provides comprehensive guidance for AI assistants working on rxiv-maker development, including coding standards, security practices, and development workflows.

## 🎯 Core Development Principles

### 1. **Code Quality First**
- Write self-documenting code with clear variable names
- Use type hints consistently throughout Python code
- Follow PEP 8 standards with ruff formatting
- Maintain comprehensive test coverage (aim for >80%)

### 2. **Security by Design**
- Validate all user inputs at boundaries
- Use parameterized queries for database operations
- Sanitize file paths and shell commands
- Follow principle of least privilege

### 3. **Performance Awareness**
- Profile before optimizing
- Use appropriate data structures
- Cache expensive operations
- Consider memory usage for large files

## Development Environment Setup

### Initial Setup
- **NEVER install packages at system level**
- **ALWAYS use the virtual environment (.venv)**
- **ALWAYS use UV for package management** (fast and modern)

```bash
# Recommended setup with UV (primary method)
uv pip install -e ".[dev]"
pre-commit install

# Legacy method (only if UV unavailable)
pip install -e ".[dev]"
```

### Development Dependencies
Install development dependencies for full functionality:
```bash
# Full development environment (UV - recommended)
uv pip install -e ".[dev]"  # Includes pytest, ruff, mypy, pre-commit, PyPDF2, etc.

# Or install specific packages with UV
uv pip install pytest pytest-cov pytest-xdist ruff mypy pre-commit hatch build PyPDF2
```

## Testing Commands

### Primary Testing (Nox-based - Recommended)
The project uses Nox for automated testing workflows:

```bash
# Main testing commands
nox -s test                    # Full test suite (default)
nox -s test-unit              # Unit tests only (fastest)
nox -s test-integration       # Integration tests only
nox -s test-fast              # Fast tests for development
nox -s test-smoke             # Ultra-fast smoke tests
nox -s test-system            # System tests (manual trigger)

# Cross-platform testing
nox -s test_cross             # Test across Python versions
nox -s test_cli_e2e          # End-to-end CLI testing with package build
```

### Local Testing
```bash
# Test with local engine (only supported engine)
nox -s pdf                    # Test PDF generation (local)
```

### Direct pytest Commands
```bash
# Direct pytest usage
pytest tests/unit/ -v                    # Unit tests
pytest tests/integration/ -v             # Integration tests
pytest tests/system/ -v                  # System tests
pytest tests/ -m "fast"                  # Fast tests only
pytest tests/ -m "not slow"              # Exclude slow tests
pytest tests/ --maxfail=3               # Stop after 3 failures

# Coverage testing
pytest --cov=src/rxiv_maker tests/
```

### Legacy Commands (Still Supported)
```bash
# rxiv CLI commands
rxiv pdf                      # Generate PDF from manuscript
rxiv clean                    # Clean output files
rxiv check-installation      # Verify installation
```

## Code Quality Commands

### Linting and Formatting
```bash
# Nox-based (recommended)
nox -s lint                   # Run linting checks
nox -s format                 # Format code (auto-fix)

# Direct ruff usage
ruff check src/ tests/        # Lint code
ruff format src/ tests/       # Format code
ruff check --fix src/         # Lint with auto-fix
```

### Type Checking
```bash
# Type checking with mypy
mypy src/rxiv_maker
nox -s type-check            # Via nox (if available)
```

### Security Scanning
```bash
nox -s security              # Security vulnerability scanning
```

## Build and Development Commands

### Package Building
```bash
# Build package
nox -s build                 # Full build and validation
hatch build                  # Build with hatch
python -m build              # Standard build

# Test installation
pip install dist/*.whl       # Install built wheel
```

### Development Validation
```bash
# Check installation and dependencies
rxiv check-installation      # Verify setup
rxiv --version               # Check version
rxiv --help                  # Show help

# Validate manuscript
rxiv validate MANUSCRIPT/    # Validate manuscript structure
```

## Repository Management

rxiv-maker supports managing multiple manuscript repositories with GitHub integration.

### Initial Setup
```bash
# Interactive setup
rxiv repo-init

# Manual configuration
rxiv config set-repo-parent-dir ~/manuscripts
rxiv config set-repo-org HenriquesLab
rxiv config set-repo-editor code
```

### Repository Commands
```bash
# Create new manuscript repository (manuscript-{name} prefix)
rxiv create-repo my-paper            # Local only
rxiv create-repo my-paper --github   # With GitHub repo creation

# List all repositories with status
rxiv repos                           # Shows git status, uncommitted changes

# Search and clone from GitHub
rxiv repos-search my-paper           # Interactive search/clone
```

### Configuration Management
```bash
# Interactive configuration menu (default)
rxiv config

# Non-interactive mode (show current settings)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HenriquesLab/rxiv-maker](https://github.com/HenriquesLab/rxiv-maker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
