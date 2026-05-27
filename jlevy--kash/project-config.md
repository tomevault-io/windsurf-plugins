---
trigger: always_on
description: Python Coding Guidelines
---

# Python Coding Guidelines

These are rules for a modern Python project using uv.

## Python Version

Write for Python 3.11-3.13. Do NOT write code to support earlier versions of Python.
Always use modern Python practices appropriate for Python 3.11-3.13.

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
  make install
  
  # Run linting (with ruff) and type checking (with basedpyright).
  # Note when you run this, ruff will auto-format and sort imports, resolving any
  # linter warnings about import ordering:
  make lint
  
  # Run tests:
  make test
  
  # Run uv sync, lint, and test in one command:
  make
  ```

- The usual `make test` like standard pytest does not show test output.
  Run individual tests and see output with `uv run pytest -s some/file.py`.

- Always run `make lint` and `make test` to check your code after changes.

- You must verify there are zero linter warnings/errors or test failures before
  considering any task complete.

## General Development Practices

- Be sure to resolve the pyright (basedpyright) linter errors as you develop and make
  changes.

- If type checker errors are hard to resolve, you may add a comment `# pyright: ignore`
  to disable Pyright warnings or errors but ONLY if you know they are not a real problem
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

- For simple tests, prefer inline functions in the original code file below a `## Tests`
  comment. This keeps the tests easy to maintain and close to the code.
  Inline tests should NOT import pytest or pytest fixtures as we do not want runtime
  dependency on pytest.

- DO NOT write one-off test code in extra files that are throwaway.

- DO NOT put `if __name__ == "__main__":` just for quick testing.
  Instead use the inline function tests and run them with `uv run pytest`.

- You can run such individual tests with `uv run pytest -s src/.../path/to/test`

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
  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jlevy/kash](https://github.com/jlevy/kash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
