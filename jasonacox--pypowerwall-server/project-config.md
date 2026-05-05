---
trigger: always_on
description: This document provides guidelines for AI code agents (Claude, Copilot, etc.) working on this codebase.
---

# AGENTS.md - AI Agent Guidelines for pypowerwall-server

This document provides guidelines for AI code agents (Claude, Copilot, etc.) working on this codebase.

## Quick Reference

```bash
# Run tests
pytest

# Run tests with coverage
pytest --cov=app

# Run server locally
PW_EMAIL="user@example.com" ./run.sh

# Build package
python -m build

# Upload to PyPI
./upload.sh
```

## Code Style

This project follows **Black** formatting conventions:

- **Line length**: 88 characters max
- **Quotes**: Double quotes for strings (`"string"` not `'string'`)
- **Indentation**: 4 spaces (no tabs)
- **Trailing commas**: Use in multi-line structures
- **Blank lines**: 2 between top-level definitions, 1 between methods

### Import Order

```python
# 1. Standard library imports
import asyncio
import json
import logging
from datetime import datetime
from typing import Any, Dict, List, Optional

# 2. Third-party imports
import pypowerwall
from fastapi import APIRouter, HTTPException
from pydantic import BaseModel

# 3. Local imports
from app.config import settings
from app.core.gateway_manager import gateway_manager
from app.models.gateway import Gateway, GatewayStatus
```

### Type Hints

Always use type hints for function signatures:

```python
def get_gateway(self, gateway_id: str) -> Optional[GatewayStatus]:
    """Get status for a specific gateway."""
    ...

async def get_vitals() -> dict:
    """Get vitals data."""
    ...
```

## Architecture Patterns

### 1. Singleton Gateway Manager

The `GatewayManager` is a module-level singleton. Never instantiate it directly:

```python
# ✅ Correct
from app.core.gateway_manager import gateway_manager
status = gateway_manager.get_gateway("default")

# ❌ Wrong
manager = GatewayManager()
```

### 2. Cache-First Architecture

All API endpoints read from the in-memory cache populated by background polling. **Never make blocking pypowerwall calls during HTTP requests**:

```python
# ✅ Correct - read from cache
@router.get("/aggregates")
async def get_aggregates():
    status = gateway_manager.get_gateway(gateway_id)
    return status.data.aggregates or {}

# ❌ Wrong - blocking call during request
@router.get("/aggregates")
async def get_aggregates():
    pw = gateway_manager.get_connection(gateway_id)
    return pw.poll("/api/meters/aggregates")  # BLOCKS!
```

### 3. Late Imports for Circular Dependency Avoidance

Import `settings` inside functions when needed to avoid circular imports:

```python
# ✅ Correct - late import
async def _poll_gateway(self, gateway_id: str):
    from app.config import settings  # Import inside function
    if not settings.neg_solar:
        ...

# ❌ Wrong - top-level import causes circular dependency
from app.config import settings  # At module level in gateway_manager.py
```

### 4. ThreadPoolExecutor Bridge

pypowerwall is synchronous. Use the executor to avoid blocking the async event loop:

```python
# ✅ Correct
loop = asyncio.get_running_loop()
result = await asyncio.wait_for(
    loop.run_in_executor(self._executor, pw.poll, "/api/meters/aggregates"),
    timeout=10.0
)

# ❌ Wrong - blocks event loop
result = pw.poll("/api/meters/aggregates")
```

### 5. Explicit API Endpoints

Define all endpoints explicitly. **No catch-all routes**:

```python
# ✅ Correct - explicit endpoints
@router.get("/api/networks")
async def get_networks():
    ...

@router.get("/api/powerwalls")
async def get_powerwalls():
    ...

# ❌ Wrong - catch-all breaks graceful degradation
@router.get("/api/{path:path}")
async def catch_all(path: str):
    return await gateway_manager.call_api("default", "poll", f"/api/{path}")
```

### 6. Safe Defaults

Return empty collections on errors, not exceptions. This applies to **data-returning endpoints**:

```python
# ✅ Correct - safe default
if not status or not status.data:
    return {}
return status.data.vitals or {}

# ❌ Wrong - raises exception in a data endpoint
if not status:
    raise HTTPException(status_code=503, detail="Gateway offline")
```

> **Note**: The `get_default_gateway()` helper in `legacy.py` intentionally **does** raise `HTTPException(503)` when no gateway is configured — this is by design, not a violation of this pattern.

### 7. Data Transformations at Fetch Time

Apply data transformations (like `neg_solar` correction) when data is fetched, not on every request:

```python
# ✅ Correct - transform once at fetch time (in gateway_manager._poll_gateway)
if aggregates and not settings.neg_solar:
    if aggregates.get("solar", {}).get("instant_power", 0) < 0:
        aggregates["load"]["instant_power"] -= aggregates["solar"]["instant_power"]
        aggregates["solar"]["instant_power"] = 0

# ❌ Wrong - transform on every request (wasteful, requires deepcopy)
@router.get("/aggregates")
async def get_aggregates():
    aggregates = deepcopy(status.data.aggregates)  # Expensive!
    if not settings.neg_solar and solar < 0:
        ...
```

## Configuration

### Environment Variables

All configuration uses `PW_` prefix for consistency with pypowerwall proxy:

| Variable | Default | Description |
|----------|---------|-------------|
| `PW_HOST` | None | Gateway IP address |
| `PW_GW_PWD` | None | Gateway WiFi password |
| `PW_EMAIL` | None | Tesla account email |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jasonacox/pypowerwall-server](https://github.com/jasonacox/pypowerwall-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
