---
trigger: always_on
description: `terok` orchestrates and instruments containerized AI coding agents using Podman. It ships both a CLI (`terok`) and a Textual TUI (`terok-tui`). The hardened container runtime (shield, gate, SSH, podman lifecycle) is provided by the `terok-sandbox` package.
---

# Agent Guide (terok)

## Purpose

`terok` orchestrates and instruments containerized AI coding agents using Podman. It ships both a CLI (`terok`) and a Textual TUI (`terok-tui`). The hardened container runtime (shield, gate, SSH, podman lifecycle) is provided by the `terok-sandbox` package.

## Technology Stack

- **Language**: Python 3.12+
- **Package Manager**: Poetry
- **Container Runtime**: Podman
- **Testing**: pytest with coverage
- **Linting/Formatting**: ruff
- **Module Boundaries**: tach (enforced in CI via `tach.toml`)
- **Documentation**: MkDocs with Material theme
- **TUI Framework**: Textual

## Repo layout

- `src/terok/`: Python package (CLI in `src/terok/cli/`, TUI in `src/terok/tui/`)
- `tests/`: `pytest` test suite
- `docs/`: user + developer documentation
- `examples/`, `completions/`: sample configs and shell completions

## Build, Lint, and Test Commands

**Before committing:**
```bash
make lint      # Run linter (required before every commit)
make format    # Auto-fix lint issues if lint fails
```

**Before pushing:**
```bash
make test         # Run full test suite with coverage
make tach         # Check module boundary rules (tach.toml)
make lint-imports # Check cross-package import boundaries
make docstrings   # Check docstring coverage (minimum 95%)
make reuse        # Check REUSE (SPDX license/copyright) compliance
make security     # Run bandit SAST scan (no medium/high findings allowed)
make check        # Run all checks (equivalent to CI)
```

**When `pyproject.toml` changes** (added/removed/changed dependencies):

```bash
poetry lock --no-update   # Regenerate lockfile without upgrading existing deps
make install-dev          # Apply the updated lockfile to your local environment
# Commit both pyproject.toml and poetry.lock together
```

**Other useful commands:**
```bash
make install-dev  # Install all development dependencies
make docs         # Serve documentation locally
make clean        # Remove build artifacts
make spdx NAME="Real Human Name" FILES="src/terok/new_file.py"  # Add SPDX header
```

## Coding Standards

- **Style**: Follow ruff configuration in `pyproject.toml`
- **Line length**: 100 characters (ruff formatter target; `E501` is disabled so long strings that cannot be auto-wrapped are tolerated)
- **Imports**: Sorted with isort (part of ruff)
- **Type hints**: Use Python 3.12+ type hints
- **Docstrings**: Required for all public functions, classes, and modules (enforced by `docstr-coverage` at 95% minimum in CI)
- **Testing**: Add tests for new functionality; maintain coverage
- **SPDX headers**: Every source file (`.py`, `.sh`, etc.) must have an SPDX header. Use `make spdx` to add or update it — it handles both new files and existing files correctly:
  ```bash
  make spdx NAME="Real Human Name" FILES="path/to/file.py"
  ```
  - **New file** → creates the header:
    ```python
    # SPDX-FileCopyrightText: 2025 Jiri Vyskocil
    # SPDX-License-Identifier: Apache-2.0
    ```
  - **Existing file** → adds an additional copyright line (preserves the original):
    ```python
    # SPDX-FileCopyrightText: 2025 Jiri Vyskocil
    # SPDX-FileCopyrightText: 2026 New Contributor
    # SPDX-License-Identifier: Apache-2.0
    ```
  When modifying an existing file, always run `make spdx` with the contributor's name to add their copyright line. NAME must be a real person's name (ASCII-only), not a project name. Use a single year (year of first contribution), not a range. Ask the user for their name if unknown. Files covered by `REUSE.toml` glob patterns (`.md`, `.yml`, `.toml`, `.json`, etc.) do not need inline headers. `make reuse` checks compliance but does not generate headers.
- **Emojis**: Must be natively wide (`East_Asian_Width=W`) — no VS16 (U+FE0F) sequences. Use `draw_emoji()` from `terok.lib.util.emoji` for aligned output. See `docs/developer.md` → "Emoji width constraints" for details
- **No magic literals**: Never use literal IPs, URLs, ports, or filesystem paths directly in code. Define them as named constants and import from there — `tests/constants.py` for test code, appropriate module-level constants for production code. This centralises magic values and makes future changes trivial. In tests, mock filesystem paths must use a subdirectory under `MOCK_BASE` (e.g. `/tmp/terok-testing/...`) — never `/tmp` directly

## Development Workflow

1. Make changes in appropriate module (`src/terok/`)
2. Run `make lint` frequently during development
3. Add/update tests in `tests/` directory
4. Run `make test` to verify changes
5. If you added or changed cross-module imports, run `make tach` to verify module boundary rules
6. Update documentation in `docs/` if needed
7. Run `make check` before pushing

## Key Guidelines

- **Container Readiness**: When modifying init scripts or server startup, preserve readiness markers (see `docs/developer.md`)
- **Security Modes**: Understand online vs gatekeeping modes when working with git operations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [terok-ai/terok](https://github.com/terok-ai/terok) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
