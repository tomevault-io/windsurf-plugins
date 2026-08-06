---
trigger: always_on
description: **vinted_scraper** is a Python package for scraping the Vinted marketplace. It supports both synchronous and asynchronous operations with automatic cookie management and typed responses.
---

# AGENTS.md

## Project Overview

**vinted_scraper** is a Python package for scraping the Vinted marketplace. It supports both synchronous and asynchronous operations with automatic cookie management and typed responses.

- **Language**: Python (supports 3.8 – 3.14)
- **Development Python version**: 3.14 (see `.python-version`)
- **Package manager**: [uv](https://github.com/astral-sh/uv)
- **Build tool**: `uv_build`
- **HTTP client**: `httpx[brotli]`
- **License**: MIT

## Development Commands (Makefile)

All common tasks are available via the Makefile. Run `make help` for a quick reference.

| Command         | Description                                                           |
| --------------- | --------------------------------------------------------------------- |
| `make test`     | Run all unit tests (`uv run python -m unittest discover`)             |
| `make coverage` | Run tests with coverage report (source, XML, terminal)                |
| `make fmt`      | Format code: `no_implicit_optional`, `black`, `isort --profile black` |
| `make lint`     | Run Super Linter via Docker (comprehensive static analysis)           |
| `make build`    | Compile the library (`uv build`)                                      |
| `make docs`     | Generate API documentation with `pdoc`                                |
| `make clean`    | Remove build artifacts, coverage files, docs                          |
| `make all`      | Run `fmt`, `lint`, and `coverage` in sequence                         |

### Setup

```bash
uv sync   # Install all dependencies including dev group
```

### Testing

```bash
make test       # Run all unit tests
make coverage   # Run tests + generate coverage (source, XML, terminal report)
```

- Framework: **unittest** (standard library), not pytest
- Test discovery: `python -m unittest discover` from project root
- Tests live in `tests/` and import source as `from src.vinted_scraper import ...`
- Mocking: `unittest.mock` (`patch`, `MagicMock`, `AsyncMock`)
- Async tests use `unittest.IsolatedAsyncioTestCase`
- Sample data fixtures in `tests/samples/` (JSON and HTML files)
- Shared test utilities in `tests/utils/` (`_mock.py`, `_fs.py`)
- Coverage tool: `coverage` (generates `coverage.xml` and terminal report via `make coverage`)

### Formatting

```bash
make fmt
```

Runs in order:

1. `no_implicit_optional` — rewrites implicit `Optional` types
2. `black` — code formatter
3. `isort --profile black` — import sorter (black-compatible)

### Linting

```bash
make lint
```

Runs [Super Linter](https://github.com/super-linter/super-linter) v8.6.0 inside Docker. Disabled validators: `PYTHON_MYPY`, `TRIVY`, `BIOME_FORMAT`, `BIOME_LINT`, `PYTHON_RUFF`, `PYTHON_RUFF_FORMAT`. The linter also auto-fixes YAML, Markdown, JSON, Python (black/isort), and GitHub Actions files.

## Architecture

### Layer Diagram

```bash
┌──────────────────────────────────────────────────┐
│            Public API (__init__.py)              │
│  VintedScraper / AsyncVintedScraper (typed)      │
│  VintedWrapper  / AsyncVintedWrapper (raw JSON)  │
└──────────────────────────────────────────────────┘
        │                        │
        ▼                        ▼
┌──────────────────┐  ┌──────────────────────────┐
│  _scraper.py     │  │  _async_scraper.py       │
│  (typed models)  │  │  (typed models, async)   │
└──────────────────┘  └──────────────────────────┘
        │                        │
        ▼                        ▼
┌──────────────────┐  ┌──────────────────────────┐
│  _wrapper.py     │  │  _async_wrapper.py       │
│  (httpx.Client)  │  │  (httpx.AsyncClient)     │
└──────────────────┘  └──────────────────────────┘
        │                        │
        └──────────┬─────────────┘
                   ▼
        ┌──────────────────────┐
        │  _base_wrapper.py    │
        │  (shared non-I/O     │
        │   logic: validation, │
        │   headers, retry,    │
        │   cookie handling)   │
        └──────────────────────┘
                   │
                   ▼
        ┌──────────────────────┐
        │    utils/            │
        │  _constants.py       │
        │  _httpx.py           │
        │  _misc.py            │
        │  _log.py             │
        │  agents.json         │
        └──────────────────────┘
                   │
                   ▼
        ┌──────────────────────┐
        │    models/           │
        │  VintedJsonModel     │
        │  VintedItem          │
        │  VintedUser          │
        │  VintedBrand         │
        │  VintedImage         │
        │  VintedMedia         │
        │  VintedHighResolution│
        └──────────────────────┘
```

### Key Architectural Choices

1. **Wrapper vs Scraper split**
   - `VintedWrapper` / `AsyncVintedWrapper` return **raw JSON dictionaries**
   - `VintedScraper` / `AsyncVintedScraper` inherit from the wrappers and return **typed dataclass models** (`VintedItem`, `VintedJsonModel`)
   - This lets consumers choose their preferred abstraction level

2. **Shared base class (`BaseVintedWrapper`)**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Giglium/vinted_scraper](https://github.com/Giglium/vinted_scraper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
