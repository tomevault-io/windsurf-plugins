---
trigger: always_on
description: <!-- SPECKIT START -->
---

# Briefcase — Agent Development Guide

<!-- SPECKIT START -->
For additional context about technologies to be used, project structure,
shell commands, and other important information, read the current plan
<!-- SPECKIT END -->

Briefcase converts Python projects into standalone native applications for macOS, Windows, Linux, iOS, Android, and Web. This guide provides the context AI coding agents need to contribute effectively.

## Quick Reference

- **Language**: Python >= 3.10 (3.10–3.14 supported)
- **Docstrings**: Sphinx style with Markdown content
- **Dev environment**: Python 3.13 virtualenv with `dev` dependency group
- **License**: BSD-3-Clause
- **Entry point**: `briefcase` via `src/briefcase/__main__.py:main()`
- **Test framework**: pytest (100% coverage required, no exceptions)
- **Linting**: ruff (format + check), codespell, docformatter
- **Docs**: MkDocs

## Development Environment Setup

All development must use a Python 3.13 virtual environment with the `dev` dependency group installed:

```bash
python3.13 -m venv venv
source venv/bin/activate  # or venv\Scripts\activate on Windows
pip install -e . --group dev
```

This installs tox, pre-commit, and other development tooling. All test execution and CI-equivalent checks run through tox.

## Project Layout

```text
src/briefcase/
├── __main__.py          # CLI entry point
├── cmdline.py           # Command-line parsing and dispatch
├── config.py            # AppConfig / GlobalConfig from pyproject.toml
├── console.py           # Rich-based Console (ALL user I/O)
├── exceptions.py        # BriefcaseError hierarchy
├── constants.py         # Reserved words and constants
├── commands/            # Command implementations
│   ├── base.py          # BaseCommand ABC
│   ├── create.py, build.py, run.py, update.py, package.py,
│   │   publish.py, dev.py, new.py, open.py, convert.py, upgrade.py
├── platforms/           # Platform plugins (one subdir per OS)
│   ├── __init__.py      # get_platforms(), get_output_formats()
│   ├── macOS/, linux/, windows/, android/, iOS/, web/
├── integrations/        # External tool wrappers
│   ├── base.py          # Tool / ManagedTool ABCs, ToolCache
│   └── subprocess.py, android_sdk.py, docker.py, xcode.py, ...
├── bootstraps/          # App template bootstraps
├── channels/            # Publication channels (App Store, Play Store)
└── debuggers/           # Debugger plugins (pdb, debugpy)

tests/                   # Mirrors src/ structure exactly
├── conftest.py          # Root fixtures (no_print, dummy_console, configs)
├── utils.py             # DummyConsole, PartialMatchString, file helpers
├── commands/
├── platforms/
├── integrations/
└── ...

docs/en/                 # MkDocs documentation (English)
changes/                 # Towncrier changelog fragments
automation/              # Separate automation subpackage
debugger/                # Separate debugger subpackage (own pyproject.toml)
```

## Critical Rules

### No `print()` — ever

All user-facing output MUST go through the `Console` object, never raw `print()`. The `T20` ruff rule bans print statements in source code, and the `no_print` autouse test fixture will **fail any test** where briefcase code calls `print()`.

Use `self.tools.console.info()`, `.verbose()`, `.debug()`, `.warning()`, or `.error()` instead.

### 100% test coverage — no exceptions

Coverage is enforced by `coverage report --fail-under=100` in CI. Every line of new code must be covered. Platform-specific code that is unreachable on certain OSes must use conditional coverage pragmas:

```python
# pragma: no-cover-if-not-macos
# pragma: no-cover-if-is-windows
# pragma: no-cover-if-lt-py312
```

Each pragma must have a corresponding rule in `pyproject.toml` under `[tool.coverage.coverage_conditional_plugin.rules]`.

Red/Green TDD should be used when adding features or fixing bugs.

### Warnings are errors

pytest is configured with `filterwarnings = ["error"]`. Do not suppress warnings — fix the cause.

### Modifying AGENTS.md rules

Do not make changes to AGENTS.md unless specifically directed to do so.

## Architecture Patterns

### Command dispatch

1. `cmdline.parse_cmdline()` resolves platform + format from CLI args
2. Loads the format module via entry points
3. Gets the command class via `getattr(format_module, command_name)`
4. Command classes are composed via **multiple inheritance**:

```python
class macOSAppBuildCommand(
    macOSAppMixin,          # format-specific paths/behavior
    macOSSigningMixin,      # platform signing logic
    AppPackagesMergeMixin,  # utility mixin
    BuildCommand,           # base command from commands/
):
    ...
```

### Platform module conventions

Each format file (e.g., `platforms/macOS/app.py`) must export **module-level aliases** matching command names:

```python
create = macOSAppCreateCommand
update = macOSAppUpdateCommand
build = macOSAppBuildCommand
run = macOSAppRunCommand
package = macOSAppPackageCommand
publish = macOSAppPublishCommand
open = macOSAppOpenCommand
```

New platforms/formats MUST register via entry points in `pyproject.toml`, never by hard-coding in core logic.

### Tool system

- `Tool` ABC with `verify()` classmethod (calls `verify_host()` then `verify_install()`)
- `ManagedTool(Tool)` adds `exists()`, `install()`, `uninstall()`, `upgrade()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [beeware/briefcase](https://github.com/beeware/briefcase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
