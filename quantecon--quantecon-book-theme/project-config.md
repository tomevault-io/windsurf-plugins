---
trigger: always_on
description: QuantEcon Book Theme is a Sphinx theme specifically designed for Jupyter Book projects. It combines Python packaging with Node.js/webpack for asset compilation and includes comprehensive testing and documentation workflows.
---

# QuantEcon Book Theme

QuantEcon Book Theme is a Sphinx theme specifically designed for Jupyter Book projects. It combines Python packaging with Node.js/webpack for asset compilation and includes comprehensive testing and documentation workflows.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## ⚠️ CORE RULES - READ FIRST

These rules MUST be followed in every session:

1. **GitHub CLI (gh) Output**: ALWAYS write `gh` command output to a `/tmp` file for reliable capture:
   ```bash
   gh run list 2>&1 | tee /tmp/gh_output.txt
   gh pr view 123 2>&1 | tee /tmp/gh_output.txt
   ```

2. **NEVER Cancel Long-Running Commands**: Many commands take 5-15+ minutes. Set appropriate timeouts:
   - `npm install` - ~50 seconds
   - `tox` - 5-15 minutes (set timeout to 30+ minutes)
   - `tox -e docs-update` - 5-15 minutes (set timeout to 30+ minutes)
   - `pip install -e .` - 3-10 minutes (set timeout to 15+ minutes)

3. **Always Use tox for Tests**: DO NOT use `pytest` directly - always use `tox` for proper environment isolation

4. **Asset Compilation**: Run `npm run build` after any changes to files in `src/quantecon_book_theme/assets/`

## Working Effectively

### Bootstrap and Environment Setup
- **CRITICAL**: This project requires both Python 3.13+ and Node.js for full functionality
- Install required tools in this order:
  - `pip install tox` - Environment and task management
  - `pip install pre-commit` - Code quality hooks
  - `npm install` - Install webpack dependencies (~50 seconds, NEVER CANCEL)
  - `pre-commit install` - Install git hooks

### Build Commands and Timing
- **Asset compilation**: `npm run build` - Compiles CSS/JS with webpack (2.5-3 seconds, VALIDATED)
- **Theme compilation**: `tox -e compile` - Uses sphinx-theme-builder (~2-5 minutes, NEVER CANCEL. Set timeout to 10+ minutes)
- **Documentation build**: `tox -e docs-update` - Builds theme documentation (~5-15 minutes, NEVER CANCEL. Set timeout to 30+ minutes)
- **Live development**: `tox -e docs-live` - Auto-rebuilding development server (~5-10 minutes initial, NEVER CANCEL. Set timeout to 20+ minutes)
- **Install theme**: `pip install -e .` - Development installation (~3-10 minutes, NEVER CANCEL. Set timeout to 15+ minutes)

### Testing and Quality Assurance
- **Run tests**: `tox` - ALWAYS use tox for running tests (~5-15 minutes, NEVER CANCEL. Set timeout to 30+ minutes)
  - Tests run against Python 3.12 and 3.13 with Sphinx 7
  - DO NOT use `pytest` directly - always use `tox` for proper environment isolation
- **Pre-commit checks**: `pre-commit run --all-files` - All formatting and linting (~2-5 minutes, NEVER CANCEL. VALIDATED: flake8 and black work correctly)
- **Linting**: `flake8 src/` - Python linting (few seconds, VALIDATED)
- **Formatting**: `black --check src/` - Code formatting check (few seconds, VALIDATED)

### Network and Connectivity Issues
- **PyPI connectivity may fail** with timeout errors in sandboxed environments
- If `pip install` commands fail with ReadTimeoutError, document this limitation: "pip install fails due to network limitations"
- **WORKAROUND**: Use `pip install --timeout=120` for better reliability
- **DO NOT** skip commands that fail due to network issues - document them as environment limitations

## Validation Scenarios

After making changes, always test these scenarios:
- **Asset compilation**: Run `npm run build` and verify CSS/JS files are generated in `src/quantecon_book_theme/theme/quantecon_book_theme/static/`
- **Theme functionality**: Install the theme and test with a sample Jupyter Book project
- **Code quality**: Run `pre-commit run --all-files` to ensure formatting and linting pass
- **Documentation**: Run `tox -e docs-update` and verify HTML builds without errors

## Key Project Structure

### Build System Components
- `package.json` + `webpack.config.js` - Node.js asset compilation
- `pyproject.toml` - Python package configuration and dependencies
- `tox.ini` - Environment management and common tasks
- `.pre-commit-config.yaml` - Code quality automation

### Source Code Layout
```
src/quantecon_book_theme/
├── __init__.py              # Main theme module
├── assets/                  # Source CSS/JS (compiled by webpack)
├── launch.py                # Theme utilities
└── theme/                   # Sphinx theme files and compiled assets
```

### Important Files
- `src/quantecon_book_theme/assets/` - Edit SCSS and JavaScript source files here
- `src/quantecon_book_theme/theme/quantecon_book_theme/static/` - Compiled CSS/JS assets (do not edit directly)
- `tests/` - Pytest test suite with regression testing
- `docs/` - Theme documentation and examples

## Common Tasks

### Making Asset Changes
1. Edit source files in `src/quantecon_book_theme/assets/`
2. Run `npm run build` to compile changes
3. For development with auto-reload: `tox -e docs-live`
4. Always test changes with `tox -e docs-update`

### Running Tests
- **Full test suite**: `tox` - ALWAYS use tox (runs tests in Python 3.12 and 3.13 environments)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [QuantEcon/quantecon-book-theme](https://github.com/QuantEcon/quantecon-book-theme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
