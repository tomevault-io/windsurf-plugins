---
trigger: always_on
description: <!-- Project-specific only. Org-wide rules belong in ../shared/ORG.md. -->
---

# Cement

<!-- Project-specific only. Org-wide rules belong in ../shared/ORG.md. -->

> `../shared` is an optional sibling checkout. Without it, only this file applies.

## Overview

Cement is a CLI application framework built around a handler/interface pattern
with the following core concepts:

**Core Application (`cement.core.foundation.App`):**
- The main `App` class in `cement/core/foundation.py` is the central orchestrator
- Uses a Meta class pattern for configuration
- Manages lifecycle through setup(), run(), and close() methods
- Supports signal handling and application reloading

**Handler System:**
- Interface/Handler pattern where interfaces define contracts and handlers provide implementations
- Core handlers: arg, config, log, output, cache, controller, extension, plugin, template
- Handlers are registered and resolved through `HandlerManager`
- Located in `cement/core/` with corresponding modules (arg.py, config.py, etc.)

**Extensions System:**
- Extensions in `cement/ext/` provide additional functionality
- Examples: ext_yaml.py, ext_jinja2.py, ext_argparse.py, etc.
- Optional dependencies managed through pyproject.toml extras

**CLI Structure:**
- Main CLI application in `cement/cli/main.py`
- Uses CementApp class that extends core App
- Includes code generation templates in `cement/cli/templates/`

**Controllers:**
- MVC-style controllers handle command routing
- Base controller pattern in controllers/base.py files
- Support nested sub-commands and argument parsing

## Repo Structure

- `cement/core/` — interfaces and default handler implementations
- `cement/ext/` — `ext_*.py` optional extensions
- `cement/cli/` — CLI app, plus `templates/` code-generation sources
- `tests/` — mirrors the source layout
- `scripts/` — dev affordances: `audit-public-api.py`, `cli-smoke-test.sh`, `devtools.py`
- `docs/` — Sphinx documentation sources
- `.planning/` — bind-mounted from the sibling planning repo; untracked

## Commands

**Testing and Compliance:**
- `make test` - Run full test suite with coverage and PEP8 compliance
- `make test-core` - Run only core library tests
- `make comply` - Run both ruff and mypy compliance checks
- `make comply-ruff` - Run ruff linting
- `make comply-ruff-fix` - Auto-fix ruff issues
- `make comply-mypy` - Run mypy type checking
- `pdm run pytest --cov=cement tests/` - Direct pytest execution
- `pdm run pytest --cov=cement.core tests/core` - Test only core components

**Development Environment:**
- `make init` - Set up local development environment
- `pdm run cement --help` - Run the cement CLI

**Documentation:**
- `make docs` - Build Sphinx documentation

**Build and Distribution:**
- `pdm build` - Build distribution packages

**GitHub Project (github.com/datafolklabs/cement):**
- `gh issue list -R datafolklabs/cement` - List open issues
- `gh issue view <number> -R datafolklabs/cement` - View a specific issue
- `gh pr list -R datafolklabs/cement` - List open pull requests
- `gh pr view <number> -R datafolklabs/cement` - View a specific PR
- `gh pr checks <number> -R datafolklabs/cement` - View CI status for a PR
- `gh api repos/datafolklabs/cement/pulls/<number>/comments` - View PR review comments

## Practices

**Key Development Practices:**
- 100% test coverage required (pytest with coverage reporting)
- 100% PEP8 compliance enforced via ruff
- Type annotation compliance via mypy
- PDM for dependency management
- Zero external dependencies for core framework (optional for extensions)

**Extension Development:**

When working with extensions:
- Check `cement/ext/` for existing extension patterns
- Optional dependencies declared in pyproject.toml under `[project.optional-dependencies]`
- Extensions follow naming pattern `ext_<name>.py`
- Must implement proper interface contracts

**Commits:**
- Conventional Commits, subject under 72 characters, per ORG.md.
- Body lines wrapped at 78 characters when a body is genuinely needed.
- Author interactively via `make commit` (runs `pdm run cz commit`).

**Branching:**
- Branch naming: `gsd/phase-{phase}-{slug}` for phase work; otherwise a short
  `feat/...`, `fix/...`, or `docs/...` slug.
- GSD's `branching_strategy` config defaults to `none`, which keeps work on
  the current branch — it will not branch for you; branch manually before the
  first commit.

**Changelog Maintenance:**
- Update `CHANGELOG.md` phase-by-phase as work lands; do not defer to release-cut time
- Append entries to the active `## X.Y.Z - DEVELOPMENT` section using the existing buckets: **Bugs**, **Features**, **Refactoring**, **Misc**, **Deprecations**
- Each entry: one line, prefixed with `[area]` — e.g., `[ext.smtp]`, `[cli]`, `[dev]`, `[core.handler]`
- Filter out planning-artifact commits (`docs(NN.N):`, `docs(state):`, `docs(quick-...):`) — they are workflow scaffolding, not user-facing changes
- Filter out commits superseded within the same branch (revert pairs, overwrites) — only the net effect ships in the changelog
- Bucket by Conventional Commit type: `fix:` → Bugs, `feat:` → Features, `refactor:` → Refactoring, `chore:` (deps/tooling/dev-env) → Misc; structural removals (drop Python version, replace toolchain) or substantive structural reshuffles → Refactoring

## Gotchas


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [datafolklabs/cement](https://github.com/datafolklabs/cement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
