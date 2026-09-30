---
trigger: always_on
description: provides the `@deprecated` decorator for classes, functions and methods. It is a
---

# AGENTS.md

Guidance for AI coding agents working in this repository.

**Deprecated** is a small, published open-source library (`pip install Deprecated`) that
provides the `@deprecated` decorator for classes, functions and methods. It is a
Python 3.12+ project managed with **uv** (environment, lockfile) and **Hatch** (version,
quality checks, test matrix). Its only runtime dependency is `wrapt >= 1.16, < 3`.

Everything in `src/deprecated/` is **public API used by many downstream projects**: keep
changes backward compatible unless a major release is explicitly being prepared.

## Commands

```sh
uv sync                     # create .venv with the locked dev dependencies (make install)
uv run pytest               # test suite, current Python (make test)
uv run pytest tests/test_deprecated.py::test_classic_deprecated_function__warns   # a single test
uv run pytest -k sphinx     # tests matching a keyword
make cov                    # tests with coverage (terminal + htmlcov/)
make test-all               # hatch test --all: every Python x wrapt combination
WRAPT_DISABLE_EXTENSIONS=1 uv run pytest   # tests with the pure Python wrapt (run in CI)

make check                  # uv lock --check + hatch check code / fmt / types (what CI runs)
make fix                    # ruff --fix and ruff format
make docs-check             # Sphinx build, warnings are errors (what CI runs)
make docs-live              # docs preview with auto-rebuild (sphinx-autobuild)
make build                  # sdist + wheel
```

`hatch check code|fmt|types` run Ruff and mypy with the versions pinned in `pyproject.toml`
(identical to the `dev` group of `uv.lock`); `uv run ruff check .` and `uv run mypy` give the
same results from the project venv. Never edit `uv.lock` by hand: use `uv lock` / `uv add`.

## Architecture

Six modules in `src/deprecated/` (src layout, `py.typed` shipped):

- `classic.py` — `ClassicAdapter` (a `wrapt.AdapterFactory`) and the `deprecated()` decorator.
  `deprecated` accepts three call forms (`@deprecated`, `@deprecated("reason")`,
  `@deprecated(reason=..., version=..., action=..., category=..., extra_stacklevel=...)`),
  typed with `@overload`. The adapter behaves differently by target:
  - **routines** are wrapped with `wrapt.decorator` (transparent proxy, keeps the signature);
  - **classes** are *not* wrapped: their `__new__` is patched in place to emit the warning.
  - `warn()` uses `warnings.warn(..., skip_file_prefixes=SKIP_FILE_PREFIXES)` so the warning
    location always points at user code, whatever the wrapt implementation (C extension or
    pure Python) and the nesting depth. Anything touching stack levels must keep that invariant
    (tests check the reported file/line).
- `sphinx.py` — `SphinxAdapter(ClassicAdapter)` plus `versionadded`, `versionchanged` and
  `deprecated`. It rewrites the docstring by appending a `.. directive:: version` block (text
  wrapped at `line_length`); only `deprecated` also emits a warning (via the parent adapter),
  and `get_deprecated_msg` strips Sphinx roles (`:func:` ...) from the message.
- `google.py` / `numpy.py` — `GoogleAdapter` / `NumpyAdapter(ClassicAdapter)` plus
  `versionadded`, `versionchanged` and `deprecated`, built like `sphinx.py`: they add (or
  extend) a `Version added` / `Version changed` / `Deprecated` section to Google-style
  (`Header:`) or NumPy-style (hyphen-underlined header) docstrings.
- `params.py` — `deprecated_params` (alias of the `DeprecatedParams` class) warns when
  deprecated *parameters* are passed. Stacked `@deprecated_params` decorators are merged
  through the `__deprecated_params__` attribute (`_DecoratorStack`) so each parameter warns
  once. It uses `functools.wraps`, not wrapt.
- `__init__.py` — exports `deprecated` and `deprecated_params`, and holds `__version__`,
  the **single source of the version** (read by Hatch, updated with `hatch version`).

Tests live in `tests/` (`test*.py`, pytest); `tests/deprecated_params/` holds the demo
scenarios of `deprecated_params`. Documentation is Sphinx + MyST Markdown in `docs/source/`;
the scripts in `docs/source/` (`tutorial/`, `sphinx/`, `google/`, `numpydoc/`) are executed
examples whose output (including warning line numbers) is quoted in the pages, which is why Ruff does not reformat `*.md`.

## Python conventions

Enforced by `pyproject.toml` (Ruff + mypy strict); CI fails on any violation.

- Python **3.12+** syntax: PEP 695 generics (`def f[T: Deprecatable](...)`, `type X = ...`),
  built-in generics, `X | None` unions, `collections.abc` types for parameters.
- **Never** `from __future__ import annotations` (banned by a Ruff `TID` rule).
- Every function in `src/` is fully annotated (`ANN` rules, mypy `strict`,
  `warn_unreachable`). `*args: Any, **kwargs: Any` is accepted for pass-through wrappers.
  Every `# type: ignore` / `# noqa` must carry its error code and ideally a reason.
  Test code is exempt from annotation rules.
- Ruff: line length **100**, double quotes, **one import per line** (`force-single-line`),
  no `print()` outside tests and docs.
- Naming: `snake_case` functions/modules, `PascalCase` classes, `SCREAMING_SNAKE_CASE`
  module constants; private helpers prefixed with `_`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [laurent-laporte-pro/deprecated](https://github.com/laurent-laporte-pro/deprecated) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
