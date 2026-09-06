---
trigger: always_on
description: Developer and agent reference for the `ibis-typing` repository.
---

# AGENTS.md — ibis-typing

Developer and agent reference for the `ibis-typing` repository.
A Python library (and pytest plugin) for typed [Ibis](https://ibis-project.org/) dataframe expressions.

---

## Tech Stack

| Layer                | Technology                                     |
|----------------------|------------------------------------------------|
| Language             | Python ≥ 3.12                                  |
| Package manager      | `uv`                                           |
| Build backend        | `hatchling`                                    |
| Linting / formatting | `ruff`                                         |
| Type checking        | `pyright`                                      |
| Dependency auditing  | `deptry`                                       |
| Testing              | `pytest` + `hypothesis`                        |
| Coverage             | `coverage.py` (≥ 90% branch coverage enforced) |
| Core dependency      | `ibis-framework[duckdb,trino]`                 |

---

## Build & Install

```bash
make deps        # uv sync --all-extras (installs everything)
make clean       # Remove .venv and reinstall from scratch
make             # Full CI pipeline: deps → lint → test → coverage
```

---

## Lint & Format

```bash
make lint        # ruff check + ruff format --check + deptry + pyright (all checks)
make format      # uv run ruff format           (auto-fix formatting)
make fix         # uv run ruff check --fix      (auto-fix lint issues)
make fix_unsafe  # uv run ruff check --fix --unsafe-fixes
make add_noqa    # uv run ruff check --add-noqa (suppress remaining issues)
```

Individual checks:

```bash
uv run ruff check          # lint only
uv run ruff format --check # format check only
uv run deptry .            # dependency audit
uv run pyright             # type checking
```

---

## Testing

Tests are segmented by backend using pytest markers.

### Run all tests (full CI suite)

```bash
make test
```

### Run a specific category

```bash
# Unit tests only (no database backend required)
uv run coverage run -m pytest --exitfirst --durations=10 -m "not ibis"

# DuckDB integration tests
TEST_IBIS_BACKEND=duck uv run coverage run -m pytest --exitfirst --durations=10 -m "duck"

# Trino integration tests (uses testcontainers)
TEST_IBIS_BACKEND=trino uv run coverage run -m pytest --exitfirst --durations=10 -m "trino"
```

### Run a single test file

```bash
uv run pytest tests/test_ibis_utils.py
uv run pytest ibis_typing/hypothesis/tests/test_hypothesis_transforms.py
```

### Run a single test case

```bash
uv run pytest tests/test_ibis_utils.py::test_select_cols
uv run pytest -k "test_sums_transactions_by_month"
```

### Doctests

Every `.py` file with `>>>` examples in docstrings is automatically tested
(`--doctest-modules` is in `addopts`). Run doctests for a single module:

```bash
uv run pytest --doctest-modules ibis_typing/utils.py
```

### Pytest configuration (from `pyproject.toml`)

```toml
[tool.pytest.ini_options]
addopts = "--doctest-modules --strict-markers"
doctest_optionflags = ["ELLIPSIS"]   # '...' matches any output in doctests
markers = ["ibis", "duck", "trino"]
```

Backend markers are **applied automatically** by the pytest plugin based on
which fixtures a test uses — no manual `@pytest.mark.*` needed in most cases.

---

## Code Style

### Formatting rules (ruff)

- Line length: **88** characters
- Quotes: **double** (`"`)
- Indent: **spaces**
- Docstring code blocks are also formatted (`docstring-code-format = true`)

### Enabled rule sets (ruff lint)

`F`, `FA`, `TID`, `COM`, `C4`, `PTH`, `I` (isort), `PLE`, `PLR`, `PLW`,
`E`, `W`, `UP` (pyupgrade), `FURB`, `RUF`

Notable ignores: `E501` (line length), `COM812` (trailing comma), `PLR2004`
(magic values), `TID252` (relative imports from parent).

### Suppressing known violations

Use inline `# noqa: <code>` comments sparingly:

```python
def aggregate(...):  # noqa: PLR0913  # Too many arguments
```

---

## Imports

1. **Always add `from __future__ import annotations`** as the first line of
   every module. This enables deferred annotation evaluation (PEP 563).
2. Use **absolute imports** from the package root by default.
3. Use **relative imports** only within `ibis_typing/` itself:
   ```python
   from .ibis_adapter import IbisSchema, IbisTable
   from . import ibis_types as it
   ```
4. **Define `__all__`** in every public module.
5. Use `TYPE_CHECKING` guards for imports that are only needed by the type
   checker, not at runtime:
   ```python
   from typing import TYPE_CHECKING
   if TYPE_CHECKING:
       from .ibis_adapter import IbisSchema
   ```
6. Local imports inside function bodies are acceptable to break circular
   import cycles — add a comment explaining the reason.

---

## Types & Annotations

This project targets **Python 3.12+** and uses its newest syntax everywhere.

### Type aliases — use the `type` statement

```python
type Int64 = int | IntegerType | None
type Array[T] = Sequence[T] | ArrayType[T] | None
type TableMap[S: IbisSchema] = MutableMapping[type[S], IbisTable[S]]
```

### Generic classes — use PEP 695 syntax

```python
class IbisTable[S: IbisSchema]: ...


def fetch_table[T: IbisSchema](self, table: IbisTable[T]) -> Iterable[T]: ...
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FortnoxAB/ibis-typing](https://github.com/FortnoxAB/ibis-typing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
