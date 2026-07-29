---
trigger: always_on
description: **Analysis Date:** 2026-03-18
---

# Coding Conventions

**Analysis Date:** 2026-03-18

## Naming Patterns

**Files:**
- Use `snake_case.py` for all Python modules: `unifi_client.py`, `unifi_session.py`, `url_validator.py`
- Test files prefixed with `test_`: `test_auth.py`, `test_cache.py`, `test_crypto.py`
- Routers named by domain: `auth.py`, `config.py`, `events.py`, `devices.py`, `webhooks.py`
- Database models in `database.py` (SQLAlchemy) or `models.py` (Pydantic) per tool

**Functions:**
- Use `snake_case` for all functions: `get_gateway_info()`, `decrypt_password()`, `run_migrations()`
- Private/internal functions prefixed with underscore: `_repair_schema()`, `_normalize_timestamp()`, `_parse_legacy_ips_event()`, `_add_missing_columns()`, `_is_expired()`
- Getters use `get_` prefix: `get_settings()`, `get_database()`, `get_scheduler()`, `get_cipher()`
- Setters use `set_` prefix: `set_gateway_info()`, `set_ips_settings()`
- Boolean checkers use `is_` prefix: `is_auth_enabled()`, `_is_expired()`
- Factory functions use `create_` prefix: `create_app()`, `create_session()`

**Variables:**
- Use `snake_case` for all variables: `gateway_info`, `ips_settings`, `password_hash`
- Constants use `UPPER_SNAKE_CASE`: `CACHE_TTL_SECONDS`, `DEFAULT_REFRESH_INTERVAL`, `RETENTION_DAYS`, `RATE_LIMIT_WINDOW`
- Private module-level state prefixed with underscore: `_scheduler`, `_settings`, `_database`, `_cache`, `_sessions`

**Types/Classes:**
- Use `PascalCase` for classes: `UniFiClient`, `ThreatEvent`, `ToolkitSettings`, `SecurityHeadersMiddleware`
- SQLAlchemy models are singular nouns: `ThreatEvent`, `UniFiConfig`, `ThreatIgnoreRule`
- Pydantic models use descriptive suffixes: `ThreatEventResponse`, `WebhookCreate`, `WebhookUpdate`, `IgnoreRuleResponse`
- Pydantic CRUD pattern: `{Entity}Create`, `{Entity}Update`, `{Entity}Response`, `{Entity}ListResponse`

## Code Style

**Formatting:**
- Black formatter with line length 100 (configured in `pyproject.toml`)
- Target Python versions: 3.9, 3.10, 3.11, 3.12

**Linting:**
- Ruff linter with line length 100, target Python 3.9 (configured in `pyproject.toml`)
- mypy configured with `warn_return_any = true`, `warn_unused_configs = true`, `disallow_untyped_defs = false`

**Line Length:** 100 characters max

**Quotes:** Double quotes for strings throughout

**Trailing Commas:** Used in multi-line function calls, dicts, and lists

## Import Organization

**Order:**
1. Standard library imports (`os`, `sys`, `logging`, `datetime`, `pathlib`, `typing`)
2. Third-party imports (`fastapi`, `sqlalchemy`, `pydantic`, `aiohttp`, `bcrypt`)
3. Local application imports (`shared.config`, `shared.database`, `tools.threat_watch.models`)

**Path Style:**
- Absolute imports throughout: `from shared.config import get_settings`
- No relative imports used
- Individual items imported by name: `from sqlalchemy import select, func, desc`

**Common Import Patterns:**
```python
# Logging setup at module top
import logging
logger = logging.getLogger(__name__)

# Type hints from typing
from typing import Optional, Dict, List, Any, AsyncGenerator

# FastAPI router pattern
from fastapi import APIRouter, Depends, HTTPException, Query
router = APIRouter(prefix="/api/events", tags=["events"])

# Database session dependency
from shared.database import get_db_session
```

## Module Structure

**Every Python module starts with a docstring:**
```python
"""
Brief description of the module's purpose
"""
```

**Standard module layout:**
1. Module docstring
2. Standard library imports
3. Third-party imports
4. Local imports
5. Module-level constants
6. Module-level logger: `logger = logging.getLogger(__name__)`
7. Private state variables (prefixed with `_`)
8. Classes and functions
9. Singleton getters at bottom

## Singleton Pattern

Use module-level private variables with getter functions for singletons:

```python
# In shared/config.py
_settings: Optional[ToolkitSettings] = None

def get_settings() -> ToolkitSettings:
    global _settings
    if _settings is None:
        _settings = ToolkitSettings()
    return _settings
```

This pattern is used consistently for: `get_settings()` in `shared/config.py`, `get_database()` in `shared/database.py`, `get_scheduler()` in scheduler modules, `get_ws_manager()` in `shared/websocket_manager.py`.

## Error Handling

**API Endpoints:**
- Use `HTTPException` for client errors (400, 404):
  ```python
  raise HTTPException(status_code=404, detail="Event not found")
  raise HTTPException(status_code=400, detail="Either password or api_key must be provided")
  ```
- Catch generic exceptions, log with `exc_info=True`, return error detail:
  ```python
  except Exception as e:
      logger.error(f"Failed to save UniFi config: {type(e).__name__}: {e}", exc_info=True)
      raise HTTPException(status_code=500, detail=f"Failed to save configuration: {type(e).__name__}: {str(e)}")
  ```
- Re-raise `HTTPException` before catching generic `Exception`:
  ```python
  except HTTPException:
      raise
  except Exception as e:
      ...
  ```

**Background Tasks / Schedulers:**
- Log errors and continue (don't crash the scheduler):
  ```python
  except Exception as e:
      logger.error(f"Error delivering webhook: {e}", exc_info=True)
      return False
  ```

**Startup Errors:**
- Print to stdout with banner formatting and `sys.exit(1)`:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Crosstalk-Solutions/unifi-toolkit](https://github.com/Crosstalk-Solutions/unifi-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
