---
trigger: always_on
description: This file provides guidance to GitHub Copilot when working with code in this repository.
---

# Copilot Instructions

This file provides guidance to GitHub Copilot when working with code in this repository.

## Important Rules

- **Always write in English** - All output text (code comments, commit messages, PR descriptions, issue comments, documentation) must be in English, even if the user writes in another language.

## Task Suitability

**Good tasks for Copilot:**
- Bug fixes in existing plot implementations
- Adding new plot types following existing patterns
- Updating documentation
- Writing or improving unit tests
- Code refactoring within established patterns
- Fixing linting/formatting issues
- Updating dependencies (after checking security advisories)

**Tasks requiring human review:**
- Changes to core architecture or database schema
- Security-sensitive code (authentication, API keys, credentials)
- Complex algorithmic changes requiring domain expertise
- Breaking changes to public APIs
- Infrastructure or deployment configuration changes

**How to iterate with Copilot:**
- Use `@copilot` in PR comments to request changes or corrections
- Provide specific, actionable feedback referencing line numbers
- Link to relevant documentation or examples in the codebase
- Request explanations if the approach is unclear

## Project Overview

**anyplot** is an AI-powered platform for Python data visualization that automatically discovers, generates, tests, and maintains plotting examples. The platform is specification-driven: every plot starts as a library-agnostic Markdown spec, then AI generates implementations for all supported libraries.

**Supported Libraries** (9 total):
- matplotlib, seaborn, plotly, bokeh, altair, plotnine, pygal, highcharts, lets-plot

**Core Principle**: Community proposes plot ideas via GitHub Issues → AI generates code → AI quality review → Deployed.

## Development Setup

```bash
# Install dependencies (uses uv - fast Python package manager)
uv sync --all-extras

# Run all tests (unit + integration + e2e)
uv run pytest

# Run only unit tests
uv run pytest tests/unit

# Run only integration tests (uses SQLite in-memory)
uv run pytest tests/integration

# Run only E2E tests (requires DATABASE_URL)
uv run pytest tests/e2e

# Linting (required for CI)
uv run ruff check .

# Auto-fix linting issues
uv run ruff check . --fix

# Formatting (required for CI)
uv run ruff format .
```

### Frontend Development

```bash
cd app
yarn install
yarn dev          # Development server
yarn build        # Production build
```

## Architecture

### Plot-Centric Design

Everything for one plot type lives in a single directory:

```
plots/{spec-id}/
├── specification.md             # Library-agnostic description
├── specification.yaml           # Tags, created, issue, suggested
├── metadata/
│   └── python/                  # Per-library metadata (one file per library)
│       ├── matplotlib.yaml
│       ├── seaborn.yaml
│       └── ...
└── implementations/
    └── python/                  # Library implementations (one file per library)
        ├── matplotlib.py
        ├── seaborn.py
        └── ...
```

The `python/` subdirectory is a deliberate forward-compatibility layer; non-Python implementation languages would live as siblings (e.g. `implementations/r/`) when introduced. All paths in code, prompts, and metadata refer to the language-prefixed form: `plots/{spec-id}/implementations/python/{library}.py` and `plots/{spec-id}/metadata/python/{library}.yaml`.

Example: `plots/scatter-basic/` contains everything for the basic scatter plot.

### Spec ID Naming Convention

**Format:** `{plot-type}-{variant}-{modifier}` (all lowercase, hyphens only)

Examples: `scatter-basic`, `scatter-color-mapped`, `bar-grouped-horizontal`, `heatmap-correlation`

### Directory Structure

- **`plots/{spec-id}/`**: Plot-centric directories (spec + metadata + implementations)
- **`prompts/`**: AI agent prompts for code generation and quality evaluation
- **`core/`**: Shared business logic (database, repositories, config, utils, images)
  - **`core/database/types.py`**: Custom SQLAlchemy types (PostgreSQL + SQLite compatibility)
  - **`core/database/repositories.py`**: Data access layer
- **`api/`**: FastAPI backend (routers, schemas, dependencies, cache, analytics, MCP server)
- **`app/`**: React frontend (React 19 + TypeScript 6 + Vite 8 + MUI 9)
- **`agentic/`**: AI workflow layer (composable phases, prompt templates, runtime state)
- **`automation/`**: CI/CD helper scripts (workflow_cli, label_manager, sync_to_postgres)
- **`tests/unit/`**: Unit tests with mocked dependencies
- **`tests/integration/`**: Integration tests with SQLite in-memory database
- **`tests/e2e/`**: End-to-end tests with full FastAPI stack
- **`docs/`**: Architecture and workflow documentation

## GitHub Issue Labels

### Specification Labels

- **`spec-request`** - New specification request
- **`spec-ready`** - Specification merged to main, ready for implementations

### Implementation Labels

- **`generate:{library}`** - Trigger single library generation (e.g., `generate:matplotlib`)
- **`generate:all`** - Trigger all 9 libraries via bulk-generate
- **`impl:{library}:done`** - Implementation merged to main
- **`impl:{library}:failed`** - Max retries exhausted (3 attempts)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MarkusNeusinger/anyplot](https://github.com/MarkusNeusinger/anyplot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
