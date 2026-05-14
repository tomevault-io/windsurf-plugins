---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

<!-- AUTO-MANAGED: project-description -->
## Overview

**Act Operator** is a production-ready CLI tool that scaffolds structured LangGraph 1.0+ projects ("Act" projects) with built-in AI collaboration capabilities.

Key features:
- Generate modular LangGraph project structures via `act new`
- Add new graph components ("Casts") via `act cast`
- Upgrade skills to the latest version via `act upgrade`
- Multi-language template support (English/Korean)
- Built-in Claude Code skills for architecture, development, and testing

<!-- END AUTO-MANAGED -->

<!-- AUTO-MANAGED: build-commands -->
## Build & Development Commands

```bash
# Install dependencies
uv sync

# Run CLI directly
uv run act new              # Create new Act project
uv run act cast             # Add Cast to existing project
uv run act upgrade          # Upgrade skills in existing project

# Run tests (ignores scaffold templates)
uv run pytest

# Lint and format
pre-commit run --all-files  # Run all pre-commit hooks
ruff check --fix            # Lint with auto-fix
ruff format                 # Format code

# Build package
uv build
```

<!-- END AUTO-MANAGED -->

<!-- AUTO-MANAGED: architecture -->
## Architecture

```
act-operator/
├── act_operator/                    # Package root (contains pyproject.toml)
│   ├── pyproject.toml              # Package config (hatchling build)
│   ├── uv.lock                     # Dependency lock file
│   └── act_operator/               # Source code
│       ├── __init__.py             # Version: __version__
│       ├── __main__.py             # Entry point
│       ├── cli.py                  # Typer CLI commands (new, cast, upgrade)
│       ├── utils.py                # Name normalization, cookiecutter rendering
│       ├── version.py              # Version utilities
│       ├── scaffold/               # Cookiecutter template
│       │   ├── cookiecutter.json   # Template variables
│       │   └── {{ cookiecutter.act_slug }}/  # Template directory
│       │       ├── casts/          # Graph modules
│       │       ├── tests/          # Test structure
│       │       ├── .claude/skills/ # AI collaboration skills
│       │       └── ...
│       └── tests/                  # Unit & integration tests
├── .claude/skills/                 # CLI tool's own skills
├── .pre-commit-config.yaml         # Pre-commit hooks (ruff, uv-lock)
├── CONTRIBUTING.md                 # Contribution guide
└── README.md                       # Project documentation
```

**Data Flow:**
1. User runs `act new` → CLI prompts for act/cast names
2. `build_name_variants()` normalizes names (slug, snake, title, pascal)
3. `render_cookiecutter_template()` generates project from scaffold
4. Post-processing normalizes cast directories to snake_case

<!-- END AUTO-MANAGED -->

<!-- AUTO-MANAGED: conventions -->
## Code Conventions

- **Naming**: snake_case for functions/variables, PascalCase for classes
- **Type hints**: Required on all function signatures
- **Imports**: Sorted by ruff (isort rules), `from __future__ import annotations` first
- **Docstrings**: Google-style with Args/Returns/Raises sections
- **Private functions**: Prefixed with `_` (e.g., `_resolve_path`, `_validate_name`)
- **Constants**: UPPER_SNAKE_CASE at module level
- **Dataclasses**: Use `slots=True, frozen=True` for immutable data
- **Enums**: Use `str, Enum` base for string enums (e.g., `Language`)
- **Error handling**: Raise `typer.Exit(code=EXIT_CODE_ERROR)` for CLI errors

<!-- END AUTO-MANAGED -->

<!-- AUTO-MANAGED: patterns -->
## Detected Patterns

- **CLI Pattern**: Typer with callback for root command, subcommands via `@app.command()`
- **Name Normalization**: `NameVariants` dataclass holds slug/snake/title/pascal variants
- **Template Rendering**: Cookiecutter with post-processing for directory normalization
- **Validation**: Early validation with immediate user feedback via Rich console
- **Language Support**: `Language` enum with `from_string()` class method for flexible parsing
- **File Updates**: JSON/TOML manipulation for langgraph.json and pyproject.toml

<!-- END AUTO-MANAGED -->

<!-- AUTO-MANAGED: git-insights -->
## Git Insights

- **Main branch**: `main`
- **Commit style**: Conventional commits (fix:, feat:, chore:, etc.)
- **Pre-commit hooks**: uv-lock, ruff (lint + format)

<!-- END AUTO-MANAGED -->

<!-- AUTO-MANAGED: best-practices -->
## Best Practices

- Always run `uv sync` after modifying dependencies
- Run `pre-commit run --all-files` before committing
- Keep scaffold templates in sync with CLI logic changes
- Test CLI commands with both interactive and non-interactive modes
- Validate name inputs early to provide clear error messages

<!-- END AUTO-MANAGED -->

<!-- MANUAL -->
# Claude Code Configuration - Claude Flow V3

## Behavioral Rules (Always Enforced)

- Do what has been asked; nothing more, nothing less
- NEVER create files unless they're absolutely necessary for achieving your goal
- ALWAYS prefer editing an existing file to creating a new one
- NEVER proactively create documentation files (*.md) or README files unless explicitly requested

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Proact0/act-operator](https://github.com/Proact0/act-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
