---
trigger: always_on
description: **ALWAYS follow these instructions exactly. Only search for additional information or run exploratory bash commands if the information in these instructions is incomplete or found to be in error.**
---

# Pedal - Educational Python Code Analysis Framework

**ALWAYS follow these instructions exactly. Only search for additional information or run exploratory bash commands if the information in these instructions is incomplete or found to be in error.**

Pedal is a Python framework for analyzing student code submissions and providing educational feedback. Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Bootstrap the Environment
- Install dependencies: `pip install --upgrade pip`
- Install runtime dependencies: `pip install -r requirements.txt` -- takes ~30 seconds
- Install development dependencies: `pip install -r requirements_dev.txt` -- takes ~60 seconds  
- Install missing linting tools: `pip install flake8` -- takes ~10 seconds
- Install package in development mode: `pip install -e .` -- takes ~10 seconds

### Build and Test
- Run test suite: `python -m unittest discover -s tests/` -- takes ~3-6 seconds. NEVER CANCEL.
- Alternative test command: `make tests` -- takes ~3-6 seconds. NEVER CANCEL.
- CI test command (from tests dir): `cd tests/ && python -m unittest discover ./` -- takes ~3-6 seconds. NEVER CANCEL.
- **EXPECTED**: 16 test failures out of 521 tests exist in current repository state (path-related VPL tests and some TIFA variable analysis tests). Do not try to fix these unless directly related to your changes.
- Run with coverage: `make coverage` -- takes ~6-8 seconds. NEVER CANCEL.

### Build Documentation
- Build docs: `cd docsrc && make html` -- takes ~8 seconds. NEVER CANCEL.
- Documentation will have warnings but builds successfully
- Generated docs location: `docsrc/_build/html/`

### Code Quality
- Run linting: `flake8 pedal/ --count --exit-zero --max-complexity=10 --max-line-length=120 --statistics` -- takes ~1 second
- **EXPECTED**: ~565 style violations exist in current codebase. Only fix violations in code you modify.
- **IMPORTANT**: Linting is currently disabled in CI (.github/workflows/test_and_lint.yml) - flake8 checks are commented out
- Configuration in `setup.cfg`: max line length 120, excludes `pedal/sandbox/*` and `pedal/cait_old/*`

## Validation

### Command Line Interface Testing
ALWAYS test the pedal CLI after making changes:
```bash
# Test basic functionality
cd examples
pedal grade simple_test.py submissions/correct.py --environment terminal

# Test unit testing functionality  
pedal grade unit_test_examples.py submissions/correct.py --environment terminal

# Test help command
pedal --help
```

### Manual Validation Scenarios
ALWAYS run these scenarios after making changes to core functionality:
1. **Basic success case**: Run `pedal grade examples/simple_test.py examples/submissions/correct.py --environment terminal` - should show "Great work!" message
2. **Unit test failure case**: Run `pedal grade examples/unit_test_examples.py examples/submissions/correct.py --environment terminal` - should show failed test for `add(1,2)` expecting 5 but getting 3
3. **Sandbox mode**: Run `pedal sandbox examples/submissions/correct.py` - should execute code and show output "8"
4. **Package import**: Run `python -c "import pedal; print('Import successful')"` - should import without errors
5. **CLI help**: Run `pedal --help` - should display full command help without errors

## Key Repository Structure

### Core Modules
- **pedal/core/**: Core framework components (Report, Feedback, Environment)  
- **pedal/sandbox/**: Code execution and safety sandbox
- **pedal/assertions/**: Testing and assertion tools
- **pedal/cait/**: Code AST analysis tools
- **pedal/tifa/**: Type inference and flow analysis
- **pedal/source/**: Source code loading and parsing
- **pedal/command_line/**: CLI interface implementation

### Important Files
- **setup.py**: Package configuration, dependencies, console scripts
- **pedal/__init__.py**: Main package imports - commonly modified for new features
- **pedal/command_line/command_line.py**: CLI implementation
- **examples/**: Working examples of instructor control scripts
- **tests/**: Comprehensive test suite (521 tests)

### Documentation
- **docsrc/**: Sphinx documentation source
- **README.rst**: Package overview and installation
- **requirements.txt**: Runtime dependencies (just "tabulate")
- **requirements_dev.txt**: Development dependencies (sphinx, pytest, coverage, etc.)

## Common Tasks

### Repository Navigation - Quick Reference
When exploring the codebase, start with these key locations:
```bash
# View repository structure
ls -la
# Core package structure  
ls pedal/
# Available examples
ls examples/
# Test categories
ls tests/
# Documentation source
ls docsrc/
```

Key files to check when debugging issues:
- `pedal/__init__.py` - Main package imports and setup
- `pedal/command_line/command_line.py` - CLI entry point
- `setup.py` - Package configuration and dependencies
- `makefile` - Common development commands
- `.github/workflows/` - CI/CD pipelines

### Adding New Functionality
1. Add module to appropriate `pedal/` subdirectory

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pedal-edu/pedal](https://github.com/pedal-edu/pedal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
