---
trigger: always_on
description: feincms3 is a Django-based content management system building toolkit that provides tools and building blocks for creating versatile, powerful, and tailor-made CMS solutions for each project.
---

# feincms3 - Django CMS Building Toolkit

feincms3 is a Django-based content management system building toolkit that provides tools and building blocks for creating versatile, powerful, and tailor-made CMS solutions for each project.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Bootstrap and Setup
- Install dependencies:
  - `python -m pip install --upgrade pip tox` -- installs build tools. NEVER CANCEL: May take 2-3 minutes.
  - `pip install -e '.[all,tests]'` -- installs feincms3 in development mode with all optional dependencies and test dependencies. NEVER CANCEL: May take 5-10 minutes.
- **CRITICAL NETWORK ISSUE**: pip install commands frequently fail due to network timeouts (ReadTimeoutError from pypi.org). This is an infrastructure limitation, not a code issue. If installations fail:
  - Retry with longer timeout: `pip install --timeout=300 <package>`
  - Document as "pip install fails due to firewall/network limitations"
  - Continue with code analysis and basic Python validation

### Testing
- **CHECK TEST ENVIRONMENTS**: `tox --listenvs` -- shows all available test combinations (verified working)
- **PRIMARY TEST COMMAND**: `python -m tox -e py312-dj50` -- runs tests with Python 3.12 and Django 5.0. Takes 5-10 minutes. NEVER CANCEL. Set timeout to 15+ minutes.
  - **FREQUENT FAILURE**: This often fails during dependency installation due to network timeouts. If it fails, note "tox test fails due to network limitations during pip install phase"
- **ALTERNATIVE TEST METHODS** (when network issues prevent tox):
  - Basic import validation: `python -c "import sys; sys.path.insert(0, '.'); import feincms3; print('Import successful, version:', feincms3.__version__)"`
  - Direct Django tests: `cd tests && DJANGO_SETTINGS_MODULE=testapp.settings python manage.py test` (requires Django installation)
- Run slow tests: `python -m tox -e slowtests` -- takes 15+ minutes. NEVER CANCEL. Set timeout to 30+ minutes.

### Code Quality and Linting
- **CRITICAL**: Always run these before committing or CI will fail:
  - `python -m tox -e style` -- reformats code using ruff and runs linting checks
  - Alternative: `ruff check .` and `ruff format .` if available
- Pre-commit hooks: `pre-commit run --all-files` -- runs all code quality checks
- **RUFF CONFIG**: Project uses ruff for both linting and formatting (configured in pyproject.toml)

### Documentation
- Build docs: `python -m tox -e docs` -- builds HTML docs into `docs/build/html/`. Takes 3-5 minutes. NEVER CANCEL. Set timeout to 10+ minutes.
- Alternative: `cd docs && make html` -- if tox fails
- Documentation uses Sphinx with furo theme

### Development Workflow Commands
- **ALWAYS CHECK**: Git status and branch before making changes: `git status --porcelain && git branch -a` (verified working)
- **BASIC CODE VALIDATION**: `python -c "import sys; sys.path.insert(0, '.'); import feincms3; print('✓ Import successful, version:', feincms3.__version__)"` (verified working)
- **EXPLORE PROJECT STRUCTURE**: Key directories to examine:
  - `ls -la feincms3/` -- main package modules
  - `ls -la tests/testapp/` -- test application structure
  - `ls -la docs/` -- documentation files
  - `find . -name "*.py" -path "*/tests/*" | wc -l` -- count test files (should show ~27 test files)
- Run tests after changes: Use pytest with Django settings: `DJANGO_SETTINGS_MODULE=testapp.settings python -m pytest tests/testapp/` (requires Django installation)

## Validation Scenarios

After making changes to feincms3, **ALWAYS** validate these scenarios:
- **Core CMS functionality**: Test page creation, plugin integration, and content rendering
- **Admin interface**: Verify Django admin integration works correctly
- **Plugin system**: Test that content plugins (richtext, image, etc.) function properly
- **Tree queries**: Validate hierarchical page structure operations work correctly

## Build and Test Timing Expectations

- **CRITICAL TIMEOUT VALUES**:
  - Full test suite: 10 minutes, set timeout to 15+ minutes
  - Style/linting: 2 minutes, set timeout to 5+ minutes
  - Documentation build: 5 minutes, set timeout to 10+ minutes
  - Slow tests: 15 minutes, set timeout to 30+ minutes
  - **NEVER CANCEL any build or test command** - wait for completion

## Project Structure and Key Files

### Core Modules
- `feincms3/` -- main package directory
  - `__init__.py` -- version info (currently 5.4.2)
  - `pages.py` -- core page model and functionality
  - `applications.py` -- application integration and URL handling
  - `admin.py` -- Django admin integration
  - `plugins/` -- content plugins (richtext, image, html, etc.)
  - `renderer.py` -- content rendering system
  - `mixins.py` -- reusable model mixins

### Test Suite
- `tests/testapp/` -- test Django application
  - `settings.py` -- test configuration
  - `models.py` -- test models
  - `test_*.py` -- test files for different components
  - `manage.py` -- Django management script for tests

### Documentation
- `docs/` -- Sphinx documentation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [feincms/feincms3](https://github.com/feincms/feincms3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
