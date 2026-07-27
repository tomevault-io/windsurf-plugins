---
trigger: always_on
description: **sgqlc** is a Simple GraphQL Client for Python. It lets you declare GraphQL
---

# Copilot Instructions for sgqlc

## Project Overview

**sgqlc** is a Simple GraphQL Client for Python. It lets you declare GraphQL
schemas as Python classes, generate queries dynamically, and access results
as native Python objects — no raw GraphQL strings needed.

Key modules:

| Module | Purpose |
|--------|---------|
| `sgqlc.types` | Declare types: `Type`, `Interface`, `Enum`, `Union`, `Input`, `Field`, `Arg` |
| `sgqlc.operation` | Build and parse GraphQL operations (queries/mutations/subscriptions) |
| `sgqlc.endpoint` | Access GraphQL APIs: `HTTPEndpoint`, `RequestsEndpoint`, `HTTPXEndpoint`, `WebSocketEndpoint` |
| `sgqlc.introspection` | Introspect schemas via `python -m sgqlc.introspection` |
| `sgqlc.codegen` | Generate Python from GraphQL schemas/operations via `sgqlc-codegen` CLI |
| `sgqlc.types.datetime` | `DateTime`, `Date`, `Time` scalars (ISO 8601) |
| `sgqlc.types.relay` | `Node`, `PageInfo`, `Connection` for Relay pagination |
| `sgqlc.types.uuid` | `UUID` scalar |

## Dev Environment

The project uses **Poetry** for dependency and virtualenv management.

```bash
# Install all dependencies (including dev and all optional extras)
poetry install --all-extras --with dev

# Run any command inside the virtualenv
poetry run <command>

# Install pre-commit hooks (run once after cloning)
pre-commit install
```

## Lint, Format and Test

All quality checks run through **pre-commit**. The CI runs:

```bash
poetry run pre-commit run --hook-stage push --all-files
```

### Formatting

```bash
# Format Python code (Black, 79-char lines, single quotes)
poetry run black sgqlc/ tests/
```

Black settings (in `pyproject.toml`): line-length 79, `skip-string-normalization = true`,
targets Python 3.8–3.13. Schema/operation generated files are excluded.

### Linting

```bash
poetry run flake8 sgqlc/ tests/
```

Flake8 settings (`.flake8`): max-line-length 79, max-complexity 10.
Notable ignored rules: `I801` (isort ordering), `W503` (line break before binary
operator), `N999` (dashes in module names), `A005` (shadowing builtin names like
`http`, `uuid`, `datetime`, `types`).
Print statements are allowed only in `examples/` and `utils/`.

### Testing

```bash
# Run all tests (doctests + unit tests) with coverage
poetry run pytest

# Run a single test file
poetry run pytest tests/test-endpoint-http.py

# Run doctests for a module
poetry run pytest --doctest-modules sgqlc/types/
```

Test configuration (`pyproject.toml`):
- `testpaths = ['sgqlc', 'tests']` — both doctests and unit tests
- `--doctest-modules` is always active
- **100% coverage required** (`--cov-fail-under=100`)
- `asyncio_mode = 'auto'` (pytest-asyncio)
- Test files: `test-*.py` or `test_*.py` (kebab-case preferred)

## Code Guidelines

- Follow **The Zen of Python** (`import this`).
- **Docstrings over comments**: use reStructuredText docstrings that double as
  doctests. Inline comments should be rare and only explain non-obvious logic.
- **Prefer doctests** over unit tests — they document and test simultaneously.
  Use `tests/` files only when mocking or complex setup is needed.
- **Top-level imports** unless there is a specific reason for a local import.
- **Early returns** with the smallest branch (fewest lines of code).
- **No trailing whitespace**, no unused imports, no unused statements.
- Produce code that matches sibling functions in the same file — follow
  existing patterns exactly, do not introduce new ones.
- Use `black` to format. Use `flake8` to verify.
- Documentation: reStructuredText with `:param:`, `:type:`, `:return:`,
  `:rtype:` and cross-references (`:class:`, `:func:`, `:mod:`).
- **Single quotes** for strings (enforced by Black's `skip-string-normalization`).

## Testing Patterns

### Doctests

Most logic is tested via doctests embedded in docstrings. Example from
`sgqlc/types/__init__.py`:

```python
def has_type(self, name):
    '''Check if the type name is known.

    >>> schema = Schema()
    >>> schema.has_type('String')
    True
    '''
```

### Schema Isolation in Tests

Types registered in a `Schema` are global within that schema instance.
If you see `Schema already has XXX` errors, each test must create its own
`Schema()` and declare `__schema__ = newly_created_schema` on types defined
in that test:

```python
def test_something():
    schema = Schema()

    class MyType(sgqlc.types.Type):
        __schema__ = schema
        ...
```

### Unit Tests

Unit tests in `tests/` use pytest and follow the `test-<kebab-name>.py`
naming convention. Use `unittest.mock` or `respx` for HTTP mocking.

## Examples

Examples live in `examples/` and must always run (they are checked in CI).

```bash
# GitHub examples (require GH_TOKEN env var)
export GH_TOKEN=<token>
python3 examples/basic/01_http_endpoint.py $GH_TOKEN profusion/sgqlc

# Shopify examples (require SHOP_STORE and SHOP_TOKEN)
export SHOP_STORE=<store> SHOP_TOKEN=<token>
python3 examples/shopify/...
```

To regenerate generated schema files without live tokens:

```bash
NO_DOWNLOAD=1 ./update-schema.sh
NO_DOWNLOAD=1 ./update-operations.sh
```

**Never disclose** `GH_TOKEN`, `SHOP_STORE`, or `SHOP_TOKEN`.

When the core `sgqlc` library changes, always re-run `./update-schema.sh`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [profusion/sgqlc](https://github.com/profusion/sgqlc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
