---
trigger: always_on
description: This project was created using the
---

# AGENTS.md

## AI Code Assistant Guidelines for This Project

This project was created using the 
[Cookiecutter-Spatial-Data-Science](https://github.com/esri/cookiecutter-spatial-data-science) template, 
which is designed to streamline and promote best practices for projects combining Geography and Data Science. 
It provides a logical, reasonably standardized, and flexible project structure.

You are an AI code assistant designed to help generate and edit code for this project. Your role is to 
assist in writing clean, efficient, and well-documented code that adheres to the project's standards and 
conventions.

---

## Table of Contents

- [TL;DR — Quick Reference for Agents](#tldr--quick-reference-for-agents)
- [Project Context](#project-context)
- [Coding Standards and Conventions](#coding-standards-and-conventions)
    - [1. Coding Standards](#1-coding-standards)
    - [2. Docstring Example](#2-docstring-example)
    - [3. Markdown Conventions](#3-markdown-conventions)
    - [4. Project Structure](#4-project-structure)
    - [5. Configuration](#5-configuration)
    - [6. Dependency Management](#6-dependency-management)
    - [7. Spatial Data Science Best Practices](#7-spatial-data-science-best-practices)
    - [8. Testing Conventions](#8-testing-conventions)
    - [9. Makefile Commands and Quick Cheat Sheet](#9-makefile-commands-and-quick-cheat-sheet)
    - [10. AI Assistant Usage Guidelines](#10-ai-assistant-usage-guidelines)
    - [11. Documentation Best Practices](#11-documentation-best-practices)
    - [12. Logging Best Practices](#12-logging-best-practices)
    - [13. Notebook Hygiene](#13-notebook-hygiene)

---

## TL;DR — Quick Reference for Agents

The rules most often violated. The full rationale is below.

- **Style**: PEP 8, 4-space indent, type hints on every function/method (no `from __future__ import annotations`), Google-style docstrings.
- **Paths**: Use `pathlib.Path`, never `os.path` or string concatenation.
- **Logging**: Use a module-level `logger` from `get_logger(__name__, ...)`. No `print()` outside script entry points.
- **Config**: Read settings via `config.*` and `secrets.*` singletons. Never hardcode paths, WKIDs, or credentials.
- **Secrets**: Never commit `config/secrets.yml`. Add new keys to `secrets_template.yml` too.
- **Data**: `data/raw/` is read-only. Write outputs to `data/interim/` or `data/processed/`.
- **ArcPy**: Call as `arcpy.toolbox.Toolname(...)` with named parameters. Use `with_temp_fgdb` for large intermediates.
- **SQL/DuckDB**: Externalise non-trivial queries to `src/{{cookiecutter.support_library}}/sql/*.sql`. Always use bind parameters — never f-string interpolation.
- **Tests**: Mirror `src/` layout in `testing/`. One behavior per test. Use fixtures from `conftest.py`.
- **Scope**: Don't refactor unrelated code. Don't rename or change signatures of public symbols without approval.
- **Dependencies**: Don't add packages to `pyproject.toml` or `environment.yml` without flagging the addition.

---

## Project Context

When working in this project:

- The primary Python package is `{{cookiecutter.support_library}}`, located in `src/{{cookiecutter.support_library}}/`
- ArcGIS Pro toolboxes are in `arcgis/` and reference `{{cookiecutter.support_library}}` for shared logic
- Tests live in `testing/` and follow the naming convention `test_*.py`
- Configuration is loaded from `config/config.yml`; secrets belong in `config/secrets.yml` (gitignored)

---

## Coding Standards and Conventions

Please follow these standards and conventions when generating or editing code:

<!-- copilot:instruction name="python-style" applyTo="**/*.py,**/*.pyt" -->

### 1. Coding Standards

- **PEP 8**: All Python code must comply with [PEP 8](https://peps.python.org/pep-0008/).
- **Formatting**: Code is formatted with `black` (line length 100). Run `black src testing scripts`
  before committing.
- **Type Hints**: All functions and class methods must include explicit type hints for arguments
  and return values. Style decisions for this project:
    - Do **not** add `from __future__ import annotations` — it causes compatibility issues with
      some workflows and is not required.
    - Use PEP 604 union syntax: `int | None`, not `Optional[int]` or `Union[int, None]`.
    - Use built-in generics: `list[str]`, `dict[str, int]`, not `List[str]` / `Dict[str, int]`.
    - For ArcPy paths, annotate as `str | os.PathLike[str]` when both are accepted, else `str`.
- **Docstrings**: Use [Google-style](https://google.github.io/styleguide/pyguide.html) docstrings
  with `Args:`, and `Returns:` / `Raises:` sections when applicable. See §2 for an example and
  §3 for admonition syntax used inside docstrings.
- **Code samples in docstrings**: Do not use the `Example:` section header. Use a fenced code
  block with the `python` language tag instead.
- **Think before coding**: State your assumptions out loud. If the request is ambiguous, ask. If 
  a simpler approach exists, push back. Stop when you are confused, name what is unclear, do not 
  just pick one interpretation and run.
- **Simplicity first**: Write the minimum code that solves the problem. No speculative abstractions. 
  No flexibility nobody asked for. The test: would a senior engineer call this overcomplicated.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Esri/cookiecutter-spatial-data-science](https://github.com/Esri/cookiecutter-spatial-data-science) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
