---
trigger: always_on
description: FastAPI versioned API layout, router composition, and per-feature directory structure
---


# API Router Structure

This document defines the FastAPI layout for versioned APIs.

Use this structure when refactoring an existing FastAPI app so routing, versioning, and application startup stay predictable.

## Intent

- Keep application startup in one place: `src/<package_name>/main.py`
- Keep API version boundaries explicit under `src/<package_name>/api/`
- Keep router registration centralized in each version's `api.py`
- Keep feature endpoints inside version-local `routers/`
- Let a top-level feature own both its own endpoints and nested sub-feature routers
- Make it easy to run `v1` and `beta` side by side during migrations
- Keep versioned HTTP contracts at the edge; share business logic in `core`

## Canonical Layout

```text
src/<package_name>/
├── main.py
├── core/ # shared business logic services, standard models, and dependencies
│   ├── services/
│   ├── standard_models/
│   └── dependencies/
└── api/
    ├── __init__.py
    ├── beta/
    │   ├── __init__.py          # exports api_router
    │   ├── api.py
    │   └── routers/
    │       ├── __init__.py
    │       └── <feature>/
    │           ├── __init__.py
    │           ├── <feature>_router.py
    │           ├── <feature>_models.py
    │           ├── <feature>_service.py
    │           ├── <feature>_dependencies.py
    │           ├── <feature>_adapter.py
    │           └── sub_routers/
    │               ├── __init__.py
    │               └── <sub_feature>/
    │                   ├── __init__.py
    │                   ├── <sub_feature>_router.py
    │                   ├── <sub_feature>_models.py
    │                   ├── <sub_feature>_service.py
    │                   ├── <sub_feature>_dependencies.py
    │                   └── <sub_feature>_adapter.py
    └── v1/
        ├── __init__.py          # exports api_router
        ├── api.py
        └── routers/
            ├── __init__.py
            └── <feature>/
                ├── __init__.py
                ├── <feature>_router.py
                ├── <feature>_models.py
                ├── <feature>_service.py
                ├── <feature>_dependencies.py
                ├── <feature>_adapter.py
                └── sub_routers/
                    ├── __init__.py
                    └── <sub_feature>/
                        ├── __init__.py
                        ├── <sub_feature>_router.py
                        ├── <sub_feature>_models.py
                        ├── <sub_feature>_service.py
                        ├── <sub_feature>_dependencies.py
                        └── <sub_feature>_adapter.py
```

Every feature router gets a directory. No flat files for routers. A top-level feature may also contain its own nested `sub_routers/` directory for sub-features. The `<feature>` and `<sub_feature>` templates show the full scope of files a router package may include; omit any file that is not needed for a given feature.

## Directory Responsibilities

### `src/<package_name>/main.py`

Own the application lifecycle and top-level app wiring only.

- Create the FastAPI `app`
- Define the lifespan context manager
- Register middleware
- Register exception handlers
- Mount static assets if needed
- Include version routers by importing `api_router` from version packages
- Define only non-versioned endpoints (e.g. `/`)

Do not define versioned endpoints (e.g. `/api/v1/...` or `/api/beta/...`) in `main.py`. All versioned HTTP surface belongs under `src/<package_name>/api/<version>/...`.

Do not spread app startup, middleware setup, or version registration across multiple files.

Minimal pattern:

```python
from contextlib import asynccontextmanager
from collections.abc import AsyncGenerator

from fastapi import FastAPI

from <package_name>.api import beta
from <package_name>.api import v1


@asynccontextmanager
async def lifespan(app: FastAPI) -> AsyncGenerator[None, None]:
    # Initialize shared resources here.
    yield
    # Tear down shared resources here.


app = FastAPI(lifespan=lifespan)

app.include_router(v1.api_router)
app.include_router(beta.api_router)
```

### Version Package Exports

`src/<package_name>/api/v1/__init__.py` and `src/<package_name>/api/beta/__init__.py` export `api_router` directly so `main.py` imports the version package contract, not internal `api.py` paths.

```python
# api/v1/__init__.py
from <package_name>.api.v1.api import api_router

__all__ = ["api_router"]
```

```python
# api/beta/__init__.py
from <package_name>.api.beta.api import api_router

__all__ = ["api_router"]
```

### `src/<package_name>/api/`

This directory is the versioning boundary for the HTTP API.

- `api/beta/` is the first pass at an API surface before contracts and behavior are hardened
- `api/v1/` contains the hardened, stable versioned routes
- Future versions follow the same pattern: `api/v2/`, `api/v3/`, and so on

The top-level `api/` directory should not hold endpoint logic directly. It only organizes versions.

### `src/<package_name>/api/<version>/api.py`

This file is the composition point for a single API version.

- Define the version prefix once
- Import router modules from that version's `routers/` directory
- Register routers declaratively via a `ROUTER_MODULES` tuple and loop

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [albusOS/sku-ops](https://github.com/albusOS/sku-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
