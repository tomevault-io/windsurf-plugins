---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Django-Tailwind is a Python package that integrates Tailwind CSS with Django applications. It provides management commands, template tags, and utilities to streamline Tailwind CSS development within Django projects.

The package supports two installation modes:
- **npm-based:** Traditional approach using Node.js and npm for maximum flexibility (plugins, PostCSS, etc.)
- **Standalone binary:** Simplified approach using the Tailwind CSS standalone binary via pytailwindcss (no Node.js required)

## Development Commands

### Testing
```bash
# Run all tests
uvx --with tox-uv tox

# Run tests with pytest directly
uv run pytest

# Run specific test file
uv run pytest tests/test_cli.py
```

### Code Quality
```bash
# Format code with ruff
uv run ruff format

# Check and fix code issues
uv run ruff check --fix

# Run pre-commit on all files
pre-commit run --all-files
```

### Documentation
```bash
# Build documentation
make docs
```

### Package Management
```bash
# Install dependencies
uv sync

# Update dependencies
uv lock --upgrade

# Build package
uv build
```

### Example App Development
The `example/` directory contains a Django project with Tailwind setup:

```bash
# Navigate to example directory
cd example/

# Install Node.js dependencies for Tailwind
python manage.py tailwind install

# Start Tailwind development server (watches for changes)
python manage.py tailwind start

# Start both Django server and Tailwind watcher simultaneously
python manage.py tailwind dev

# Build production CSS
python manage.py tailwind build
```

## Architecture

### Core Components

- **Management Command** (`src/tailwind/management/commands/tailwind.py`):
  - Main entry point for all Tailwind operations
  - Handles `init`, `install`, `build`, `start`, `dev`, `check-updates`, `update`, `plugin_install` commands
  - Uses cookiecutter templates for app initialization
  - `dev` command runs Django server and Tailwind watcher simultaneously using Honcho
  - Automatically detects and routes between npm-based and standalone binary modes
  - Standalone detection checks for `package.json` presence or `TAILWIND_USE_STANDALONE_BINARY` setting

- **NPM Integration** (`src/tailwind/npm.py`):
  - Wrapper around npm commands
  - Handles subprocess execution for Node.js operations

- **Template Tags** (`src/tailwind/templatetags/tailwind_tags.py`):
  - `{% tailwind_css %}` - Includes Tailwind CSS in templates
  - `{% tailwind_preload_css %}` - Preloads CSS for performance
  - Automatically appends cache-busting parameters in DEBUG mode

- **Utilities** (`src/tailwind/utils.py`):
  - Helper functions for path resolution
  - Package.json manipulation
  - Dynamic pip package installation

### App Templates

Three cookiecutter templates are provided:
- `app_template_v3/` - For Tailwind CSS v3 (npm-based)
- `app_template_v4/` - For Tailwind CSS v4 (npm-based)
- `app_template_v4_standalone/` - For Tailwind CSS v4 standalone binary

**npm-based templates** (`v3` and `v4`) generate Django apps with:
- `static_src/` directory for source files
- `package.json` with build scripts
- PostCSS configuration
- Base HTML template

**Standalone template** (`v4_standalone`) generates minimal Django apps with:
- `static_src/src/` directory with styles.css
- No `package.json` or npm dependencies
- No PostCSS configuration
- Base HTML template
- Pre-created `static/` directory structure

### Configuration

The package uses Django settings for configuration:
- `TAILWIND_APP_NAME` - Name of the Tailwind app
- `TAILWIND_CSS_PATH` - Path to compiled CSS
- `TAILWIND_DEV_MODE` - Development mode flag (deprecated)
- `NPM_BIN_PATH` - Custom npm binary path (npm-based only)
- `TAILWIND_USE_STANDALONE_BINARY` - Force standalone binary mode (default: False, auto-detected)
- `TAILWIND_STANDALONE_BINARY_VERSION` - Tailwind CSS standalone binary version (default: "v4.3.0")

### Standalone Binary Implementation

The standalone binary mode uses [pytailwindcss](https://github.com/timonweb/pytailwindcss) to run the Tailwind CSS standalone binary without requiring Node.js.

**How it works:**
1. Detection happens in the management command via `self.is_standalone` flag
2. Checks for `TAILWIND_USE_STANDALONE_BINARY` setting OR absence of `package.json`
3. Routes command execution to either npm methods or standalone methods

**Commands behavior:**
- `init --tailwind-version 4s`: Creates standalone app using `app_template_v4_standalone`
- `install`: Downloads standalone binary via pytailwindcss (auto-install on first use)
- `build`: Runs `pytailwindcss.run()` with minify flag
- `start`: Runs `pytailwindcss.run()` with watch flag
- `dev`: Works with standalone mode (Procfile uses `tailwind start`)
- `check-updates`, `update`, `plugin_install`: Raise CommandError (not supported in standalone)

**Key implementation methods in `tailwind.py`:**
- `tailwind_cli_install_command()`: Downloads binary via pytailwindcss
- `tailwind_cli_build_command()`: Builds CSS with standalone binary
- `tailwind_cli_start_command()`: Starts watcher with standalone binary

## Common Development Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [timonweb/django-tailwind](https://github.com/timonweb/django-tailwind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
