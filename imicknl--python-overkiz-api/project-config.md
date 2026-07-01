---
trigger: always_on
description: Guidance on how to navigate and modify this codebase.
---

# Coding Agent Instructions

Guidance on how to navigate and modify this codebase.

## What this library does
It is an async Python wrapper to interact with the Overkiz API, used by vendors and platforms like Somfy TaHoma and Atlantic CozyTouch.

## Python Version

Write for Python 3.10-3.13. Do NOT write code to support earlier versions of Python.
Always use modern Python practices appropriate for Python 3.10-3.13.

Always use full type annotations, generics, and other modern practices.

## Project Setup and Developer Workflows

- Important: BE SURE you read and understand the project setup by reading the
  pyproject.toml file and the Makefile.

- ALWAYS use uv for running all code and managing dependencies.
  Never use direct `pip` or `python` commands.

- Use modern uv commands: `uv sync`, `uv run ...`, etc.
  Prefer `uv add` over `uv pip install`.

- You may use the following shortcuts
  ```shell

  # Install all dependencies:
  uv sync

  # Run linting (with ruff), prek (pre-commit alternative) checks and type checking (with mypy).
  # Note when you run this, ruff will auto-format and sort imports, resolving any
  # linter warnings about import ordering:
  uv run prek run --all-files

  # Run tests:
  uv run pytest
  ```

- Run individual tests and see output with `uv run pytest -s some/file.py`.

- You must verify there are zero linter warnings/errors or test failures before considering any task complete.

## General Development Practices

- Be sure to resolve the mypy type checker errors as you develop and make
  changes.

- If type checker errors are hard to resolve, you may add a comment `# type: ignore`
  to disable mypy warnings or errors but ONLY if you know they are not a real problem
  and are difficult to fix.

- In special cases you may consider disabling it globally it in pyproject.toml but YOU
  MUST ASK FOR CONFIRMATION from the user before globally disabling lint or type checker
  rules.

- Never change an existing comment, pydoc, or a log statement, unless it is directly
  fixing the issue you are changing, or the user has asked you to clean up the code.
  Do not drop existing comments when editing code!
  And do not delete or change logging statements.

## Coding Conventions and Imports

- Always use full, absolute imports for paths.
  do NOT use `from .module1.module2 import ...`. Such relative paths make it hard to
  refactor. Use `from toplevel_pkg.module1.modlule2 import ...` instead.

- Be sure to import things like `Callable` and other types from the right modules,
  remembering that many are now in `collections.abc` or `typing_extensions`. For
  example: `from collections.abc import Callable, Coroutine`

- Use `typing_extensions` for things like `@override` (you need to use this, and not
  `typing` since we want to support Python 3.11).

- Add `from __future__ import annotations` on files with types whenever applicable.

- Use pathlib `Path` instead of strings.
  Use `Path(filename).read_text()` instead of two-line `with open(...)` blocks.

- Use strif’s `atomic_output_file` context manager when writing files to ensure output
  files are written atomically.

## Use Modern Python Practices

- ALWAYS use `@override` decorators to override methods from base classes.
  This is a modern Python practice and helps avoid bugs.

## Testing

- For longer tests put them in a file like `tests/test_somename.py` in the `tests/`
  directory (or `tests/module_name/test_somename.py` file for a submodule).

- Don’t add docs to assertions unless it’s not obvious what they’re checking - the
  assertion appears in the stack trace.
  Do NOT write `assert x == 5, "x should be 5"`. Do NOT write `assert x == 5 # Check if
  x is 5`. That is redundant.
  Just write `assert x == 5`.

- DO NOT write trivial or obvious tests that are evident directly from code, such as
  assertions that confirm the value of a constant setting.

- NEVER write `assert False`. If a test reaches an unexpected branch and must fail
  explicitly, `raise AssertionError("Some explanation")` instead.
  This is best typical best practice in Python since assertions can be removed with
  optimization.

- DO NOT use pytest fixtures like parameterized tests or expected exception decorators
  unless absolutely necessary in more complex tests.
  It is typically simpler to use simple assertions and put the checks inside the test.
  This is also preferable because then simple tests have no explicit pytest dependencies
  and can be placed in code anywhere.

- DO NOT write trivial tests that test something we know already works, like
  instantiating a Pydantic object.

  ```python
  class Link(BaseModel):
    url: str
    title: str = None

  # DO NOT write tests like this. They are trivial and only create clutter!
  def test_link_model():
    link = Link(url="https://example.com", title="Example")
    assert link.url == "https://example.com"
    assert link.title == "Example"
  ```

## Types and Type Annotations

- Use modern union syntax: `str | None` instead of `Optional[str]`, `dict[str]` instead
  of `Dict[str]`, `list[str]` instead of `List[str]`, etc.

- Never use/import `Optional` for new code.

- Use modern enums like `StrEnum` if appropriate.

## Guidelines for Comments


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iMicknl/python-overkiz-api](https://github.com/iMicknl/python-overkiz-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
