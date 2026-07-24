---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**GeoIP2-python** is MaxMind's official Python client library for:
- **GeoIP/GeoLite Web Services**: Country, City Plus, and Insights endpoints
- **GeoIP/GeoLite Databases**: Local MMDB file reading for various database types (City, Country, ASN, Anonymous IP, Anonymous Plus, ISP, etc.)

The library provides both web service clients (sync and async) and database readers that return strongly-typed model objects containing geographic, ISP, anonymizer, and other IP-related data.

**Key Technologies:**
- Python 3.10+ (type hints throughout, uses modern Python features)
- MaxMind DB Reader for binary database files
- Requests library for sync web service client
- aiohttp for async web service client
- pytest for testing
- ruff for linting and formatting
- mypy for static type checking
- uv for dependency management and building

## Code Architecture

### Package Structure

```
src/geoip2/
├── models.py           # Response models (City, Insights, AnonymousIP, etc.)
├── records.py          # Data records (City, Location, Traits, etc.)
├── errors.py           # Custom exceptions for error handling
├── database.py         # Local MMDB file reader
├── webservice.py       # HTTP clients (sync Client and async AsyncClient)
├── _internal.py        # Internal base classes and utilities
└── types.py            # Type definitions
```

### Key Design Patterns

#### 1. **Model Classes vs Record Classes**

**Models** (in `models.py`) are top-level responses returned by database lookups or web service calls:
- `Country` - base model with country/continent data
- `City` extends `Country` - adds city, location, postal, subdivisions
- `Insights` extends `City` - adds additional web service fields (web service only)
- `Enterprise` extends `City` - adds enterprise-specific fields
- `AnonymousIP` - anonymous IP lookup results
- `AnonymousPlus` extends `AnonymousIP` - adds additional anonymizer fields
- `ASN`, `ConnectionType`, `Domain`, `ISP` - specialized lookup models

**Records** (in `records.py`) are contained within models and represent specific data components:
- `PlaceRecord` - abstract base with `names` dict and locale handling
- `City`, `Continent`, `Country`, `RepresentedCountry`, `Subdivision` - geographic records
- `Location`, `Postal`, `Traits`, `MaxMind` - additional data records

#### 2. **Constructor Pattern**

Models and records use keyword-only arguments (except for required positional parameters):

```python
def __init__(
    self,
    locales: Sequence[str] | None,  # positional for records
    *,
    continent: dict[str, Any] | None = None,
    country: dict[str, Any] | None = None,
    # ... other keyword-only parameters
    **_: object,  # ignore unknown keys
) -> None:
```

Key points:
- Use `*` to enforce keyword-only arguments
- Accept `**_: object` to ignore unknown keys from the API
- Use `| None = None` for optional parameters
- Boolean fields default to `False` if not present

#### 3. **Serialization with to_dict()**

All model and record classes inherit from `Model` (in `_internal.py`) which provides `to_dict()`:

```python
def to_dict(self) -> dict[str, Any]:
    # Returns a dict suitable for JSON serialization
    # - Skips None values and False booleans
    # - Recursively calls to_dict() on nested objects
    # - Handles lists/tuples of objects
    # - Converts network and ip_address to strings
```

The `to_dict()` method replaced the old `raw` attribute in version 5.0.0.

#### 4. **Locale Handling**

Records with names use `PlaceRecord` base class:
- `names` dict contains locale code → name mappings
- `name` property returns the first available name based on locale preference
- Default locale is `["en"]` if not specified
- Locales are passed down from models to records

#### 5. **Property-based Network Calculation**

For performance reasons, `network` and `ip_address` are properties rather than attributes:

```python
@property
def network(self) -> ipaddress.IPv4Network | ipaddress.IPv6Network | None:
    # Lazy calculation and caching of network from ip_address and prefix_len
```

#### 6. **Web Service Only vs Database Models**

Some models are only used by web services and do **not** need MaxMind DB support:

**Web Service Only Models**:
- `Insights` - extends City but used only for web service
- Simpler implementation without database parsing logic

**Database-Supported Models**:
- Models used by both web services and database files
- Must handle MaxMind DB format data structures
- Examples: `City`, `Country`, `AnonymousIP`, `AnonymousPlus`, `ASN`, `ISP`

## Testing Conventions

### Running Tests

```bash
# Install dependencies using uv
uv sync --all-groups

# Run all tests
uv run pytest

# Run specific test file
uv run pytest tests/models_test.py

# Run specific test class or method
uv run pytest tests/models_test.py::TestModels::test_insights_full

# Run tests with coverage
uv run pytest --cov=geoip2 --cov-report=html
```

### Linting and Type Checking

```bash
# Run all linting checks (mypy, ruff check, ruff format check)
uv run tox -e lint

# Run mypy type checking
uv run mypy src tests


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maxmind/GeoIP2-python](https://github.com/maxmind/GeoIP2-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
