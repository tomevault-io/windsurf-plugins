---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**minfraud-api-python** is MaxMind's official Python client library for the minFraud fraud detection services:
- **minFraud Score, Insights, and Factors**: Transaction risk assessment web services
- **Report Transaction**: API for reporting fraudulent transactions to improve detection

The library provides both synchronous and asynchronous clients that validate transaction data, communicate with the minFraud API, and return strongly-typed response models.

**Key Technologies:**
- Python 3.10+ (type hints throughout, uses modern Python features)
- Requests library for sync web service client
- aiohttp for async web service client
- voluptuous for request validation
- email_validator for email validation
- geoip2 library for IP geolocation data models (dependency)
- pytest for testing
- ruff for linting and formatting
- mypy for static type checking
- uv for dependency management and building

## Code Architecture

### Package Structure

```
src/minfraud/
├── models.py           # Response models (Score, Insights, Factors, etc.)
├── webservice.py       # HTTP clients (sync Client and async AsyncClient)
├── request.py          # Request preparation and email normalization
├── validation.py       # Request validation schemas (voluptuous)
├── errors.py           # Custom exceptions for error handling
└── version.py          # Version information
```

### Key Design Patterns

#### 1. **Client Architecture**

Two client types for different use cases:
- **`Client`** (sync) - Uses `requests` library for synchronous API calls
- **`AsyncClient`** (async) - Uses `aiohttp` for asynchronous API calls

Both clients:
- Extend `BaseClient` which handles URI construction and error handling
- Accept account ID, license key, optional host (for sandbox), locales, and timeout
- Support context manager protocol (`with` / `async with`)
- Provide methods: `score()`, `insights()`, `factors()`, `report()`

#### 2. **Request Validation and Preparation**

Two-stage process before sending to API:

**Validation** (`validation.py`):
- Uses `voluptuous` schemas to validate transaction structure
- Validates field types, formats, and value ranges
- Can be disabled by passing `validate=False` to client methods
- Raises `InvalidRequestError` if validation fails

**Preparation** (`request.py`):
- `prepare_transaction()` - Prepares Score/Insights/Factors requests
- `prepare_report()` - Prepares Report Transaction requests
- `hash_email()` - Normalizes and hashes email addresses with sophisticated logic:
  - Typo correction (e.g., `gmai.com` → `gmail.com`)
  - Equivalent domain normalization (e.g., `googlemail.com` → `gmail.com`)
  - Gmail dot removal (e.g., `f.o.o@gmail.com` → `foo@gmail.com`)
  - Yahoo/AOL alias removal (e.g., `foo-bar@yahoo.com` → `foo@yahoo.com`)
  - Fastmail subdomain handling
  - TLD typo correction (e.g., `.comcom` → `.com`)

#### 3. **Response Models**

All response models in `models.py` inherit from `_Serializable` which provides:
- `to_dict()` - Recursive serialization to plain dicts
- `__eq__()`, `__ne__()`, `__hash__()` - Comparison and hashing

**Model Hierarchy:**
- `Score` - Base response with risk score and basic info
  - Contains: `id`, `risk_score`, `funds_remaining`, `queries_remaining`, `ip_address`, `warnings`, `disposition`
- `Insights` extends `Score` - Adds detailed fraud insights
  - Adds: `credit_card`, `device`, `email`, `shipping_address`, `billing_address`, `billing_phone`, `shipping_phone`
- `Factors` extends `Insights` - Adds risk factor subscores and additional data
  - Adds: `subscores` (deprecated), `risk_score_reasons`

**Component Models:**
- `ScoreIPAddress` - Simple IP risk info for Score
- `IPAddress` extends `geoip2.models.Insights` - Full GeoIP data plus minFraud risk
- `CreditCard`, `Device`, `Email`, `EmailDomain`, `Phone` - Transaction component data
- `BillingAddress`, `ShippingAddress` - Address information
- `Disposition` - Custom rules disposition
- `ServiceWarning` - API warnings
- `IPRiskReason`, `RiskScoreReason`, `Reason` - Risk explanation objects

#### 4. **Constructor Pattern**

Models use keyword-only arguments (except for special cases):

```python
def __init__(
    self,
    *,
    field_name: str | None = None,
    is_flag: bool | None = None,
    # ... other keyword-only parameters
    **_: object,  # ignore unknown keys
) -> None:
```

Key points:
- Use `*` to enforce keyword-only arguments
- Accept `**_: object` to ignore unknown keys from the API (forward compatibility)
- Use `| None = None` for optional parameters
- Boolean fields can be `None` if not provided by API

#### 5. **Serialization with to_dict()**

All model classes inherit from `_Serializable` which provides `to_dict()`:

```python
def to_dict(self) -> dict[str, Any]:
    # Returns a dict suitable for JSON serialization
    # - Skips None values
    # - Recursively calls to_dict() on nested objects
    # - Handles lists of objects
```

#### 6. **Error Handling**

Custom exception hierarchy (all in `errors.py`):
- `MinFraudError` (base) - Generic errors
  - `AuthenticationError` - Invalid credentials

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maxmind/minfraud-api-python](https://github.com/maxmind/minfraud-api-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
