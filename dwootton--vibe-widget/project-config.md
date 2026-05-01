---
trigger: always_on
description: > Comprehensive guide for AI agents working in this codebase. Last updated: 2026-01-07
---

# Agent Guide for vibe-widget

> Comprehensive guide for AI agents working in this codebase. Last updated: 2026-01-07

## Project Overview

**vibe-widget** is a Python library that generates interactive Jupyter notebook widgets from natural language descriptions using LLMs. It combines Python backend (anywidget + traitlets) with a React-based JavaScript frontend bundled with esbuild.

- **Tech Stack**: Python 3.9+, React 19, anywidget, esbuild, OpenRouter API
- **Purpose**: Create interactive data visualizations in notebooks without writing frontend code
- **Architecture**: Hybrid Python/JavaScript with LLM-powered code generation

## Essential Commands

### Development Setup

```bash
# Install JavaScript dependencies
npm install

# Install Python package in editable mode (from repo root)
python -m pip install -e .

# Install dev dependencies
python -m pip install -e ".[dev]"
```

### Build & Watch

```bash
# Build JavaScript bundle once
npm run build-app-wrapper

# Watch mode (recommended during development)
npm run watch-app-wrapper

# Build output: src/vibe_widget/AppWrapper/AppWrapper.js → AppWrapper.bundle.js
```

**CRITICAL**: After JS changes, you MUST:
1. Rebuild the bundle (or use watch mode)
2. **Restart the Jupyter kernel** (the bundle is cached)

### Agent Done Checklist (always)
- If you touched JS: run `npm run build-app-wrapper` before handing off, or state why it wasn’t run.
- For substantial changes (backend or frontend): run the relevant tests (`pytest`, `npm run test:ui`) and report results, or state why they weren’t run.
- If a command can’t be run (env/time), say so explicitly in the final message.

### Testing

```bash
# Run full test suite
pytest



# Optional test suites (disabled by default)
RUN_PERF=1 pytest -m performance
RUN_E2E=1 pytest -m e2e

# JavaScript tests
npm run test:ui
```

### Code Quality

```bash
# Linting (ruff)
ruff check src/ tests/

# Type checking (mypy)
mypy src/

# Formatting (ruff)
ruff format src/ tests/
```

**Ruff config** (pyproject.toml):
- Line length: 100
- Target: Python 3.9
- Auto-imports sorting (isort)
- Quote style: double quotes
- Indent style: spaces

## Project Structure

### Python Source (`src/vibe_widget/`)

```
src/vibe_widget/
├── __init__.py              # Public API exports
├── api.py                   # Output/input/action API helpers (ExportHandle, bundles)
├── config.py                # Config, model catalog, OpenRouter model fetching
├── themes.py                # Theme definitions and generation
├── debug.py                 # Debug utilities
├── models_manifest.json     # Available LLM models catalog
│
├── core/                    # Core widget implementation
│   ├── widget.py           # VibeWidget class (anywidget subclass)
│   ├── state.py            # StateManager for execution/audit state
│   └── __init__.py         # create(), edit(), load(), clear()
│
├── llm/                     # LLM provider abstraction
│   ├── agentic.py          # AgenticOrchestrator (main generation flow)
│   ├── providers/
│   │   ├── base.py         # LLMProvider abstract base
│   │   └── openrouter_provider.py  # OpenRouter implementation
│   └── tools/              # Tools for agentic workflows
│       ├── base.py
│       ├── code_tools.py   # CodeValidateTool
│       ├── data_tools.py   # DataLoadTool, DataProfileTool, DataWrangleTool
│       └── execution_tools.py  # RuntimeTestTool, ErrorDiagnoseTool
│
├── services/                # Service layer
│   ├── generation.py       # GenerationService (orchestrates code generation)
│   ├── audit.py            # AuditService (security audits)
│   └── theme.py            # ThemeService
│
├── utils/                   # Utilities
│   ├── audit_store.py      # AuditStore (persistent audit cache)
│   ├── code_parser.py      # CodeStreamParser, RevisionStreamParser
│   ├── logging.py          # Logger setup
│   ├── serialization.py    # JSON serialization helpers
│   ├── util.py             # Data loading, summarization, cleaning
│   ├── validation.py       # Input name sanitization
│   └── widget_store.py     # WidgetStore (persistent widget cache)
│
└── AppWrapper/              # JavaScript/React frontend
    ├── AppWrapper.js       # Main React component
    ├── components/         # React components
    │   ├── SandboxedRunner.js
    │   ├── FloatingMenu.js
    │   ├── LoadingOverlay.js
    │   ├── SelectionOverlay.js
    │   ├── EditPromptPanel.js
    │   ├── AuditNotice.js
    │   ├── ProgressMap.js
    │   ├── DebuggerPanel.js
    │   └── editor/         # Code editor components
    │       ├── EditorViewer.js
    │       ├── CodeEditor.js
    │       ├── MessageEditor.js
    │       └── AuditPanel.js
    ├── hooks/              # React hooks
    │   ├── useModelSync.js
    │   └── useKeyboardShortcuts.js
    └── utils/              # JS utilities
```

### Tests (`tests/`)

```
tests/
├── conftest.py              # Pytest fixtures (mock LLM provider, sample data)
├── utils.py                 # Test utilities
├── mocks/                   # Mock implementations
├── fixtures/                # Test fixtures
├── unit/                    # Fast unit tests (no network)
├── integration/             # Integration tests (mocked LLM)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dwootton/vibe-widget](https://github.com/dwootton/vibe-widget) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
