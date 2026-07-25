---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Rio is a Python-first web framework for building websites and apps entirely in Python (no HTML/CSS/JavaScript required). It uses a React-like component model with a hybrid architecture:
- **Python backend**: Component definitions, logic, state management, server communication
- **TypeScript frontend**: DOM rendering, event handling, client interactions
- **Communication**: WebSocket/HTTP RPC between Python and TypeScript

## Development Commands

### Initial Setup
```bash
# Install Python dependencies (required first)
uv sync --all-extras

# Install frontend dependencies
npm install

# Build frontend (REQUIRED before first use and after TypeScript changes)
npm run build

# Install pre-commit hooks
python -m pre_commit install
```

### Building
```bash
# Development build (default)
npm run dev-build

# Production build (for releases)
npm run build
```

**When to rebuild frontend**: After any changes to TypeScript/SCSS files in `/frontend/`, before testing, and before releases.

### Testing
```bash
# Run all Python tests
uv run pytest

# Run specific test file
uv run pytest tests/test_components.py

# Run with coverage
uv run scripts/code_coverage.py

# Frontend integration tests (requires browser setup)
uv run pytest tests/test_frontend/
```

### Code Quality
```bash
# Lint Python code
python -m ruff check .

# Format Python code
python -m ruff format .

# Format TypeScript code
npx prettier --write frontend/

# Run all pre-commit hooks
pre-commit run --all-files
```

### Running Rio Apps
```bash
# Create new project from template
rio new

# Run project in development mode
rio run

# Run in browser vs window
app.run_in_browser()  # Web app
app.run_in_window()   # Local desktop app
```

## Architecture

### Component System

Rio uses a dual implementation pattern for components:

**Python Side** (`/rio/components/`):
- Component class definitions with state/logic
- Observable properties that auto-trigger re-renders on change
- `build()` method for high-level components (compose from other components)
- `_custom_serialize_()` for sending state to frontend
- `_on_message_()` for handling frontend events

**TypeScript Side** (`/frontend/code/components/`):
- `ComponentBase` subclasses that create/update DOM elements
- `createElement()`: Build initial DOM structure
- `updateElement(deltaState)`: Apply state changes from Python
- Event handlers that send messages back to Python

**Component Hierarchy**:
```
Component (metaclass: ComponentMeta)
├── FundamentalComponent (maps to TypeScript/HTML, low-level)
│   └── Examples: Button, Text, ListView, Image
└── User Components (high-level, composed via build())
    └── Examples: Custom business logic components
```

### Observable Property System

Rio uses `ComponentMeta` metaclass to transform component classes into observable dataclasses:
- Properties automatically track dependencies
- Reading a property during `build()` registers a dependency
- Changing a property triggers rebuild of dependent components
- State synchronization happens automatically between Python ↔ TypeScript

### Session Management

- Each client connection = one `Session` instance (`rio/session.py`)
- Session maintains: component tree, active page, user settings, timezone, window dimensions
- Components created within session context get auto-registered
- WebSocket communication for real-time updates

## Project Structure

### Python Backend
- `/rio/components/` - Built-in component implementations
- `/rio/app.py` - Core `App` class and application setup
- `/rio/session.py` - Session state and lifecycle management
- `/rio/cli/` - Command-line interface (rio new, rio run, etc.)
- `/rio/observables/` - Observable property system and state tracking
- `/rio/transports/` - WebSocket and HTTP transport layers
- `/rio/component_meta.py` - Metaclass for component creation

### TypeScript Frontend
- `/frontend/code/components/` - Component rendering implementations
- `/frontend/code/rpc.ts` - RPC communication with Python backend
- `/frontend/code/componentManagement.ts` - Component lifecycle and tree management
- `/frontend/css/components/` - Component-specific SCSS styles
- `/frontend/index.html` - Frontend entry point

### Build System
- `/vite.config.mjs` - Vite build configuration
- Output: `/rio/frontend files/` (auto-generated, included in wheel)
- Build process compiles TypeScript + SCSS → bundled assets

### Testing
- `/tests/test_components.py` - Component behavior tests
- `/tests/test_frontend/` - Browser-based integration tests
- `/tests/test_cli/` - CLI command tests
- `/tests/test_observables.py` - State management tests

### Scripts
- `/scripts/build_icon_set.py` - Build icon sets from SVGs
- `/scripts/code_coverage.py` - Generate coverage reports
- `/scripts/publish.py` - Release/publishing automation
- `/scripts/generate_stubs.py` - Type stub generation

## Development Workflow

### Adding New Built-in Components

1. Create Python component in `/rio/components/my_component.py`
2. Create TypeScript component in `/frontend/code/components/myComponent.ts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rio-labs/rio](https://github.com/rio-labs/rio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
