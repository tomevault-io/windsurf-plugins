---
trigger: always_on
description: Guidance for AI agents (including Claude Code) working in this repository.
---

# AGENTS.md
Guidance for AI agents (including Claude Code) working in this repository.

## Project Overview

FastAPI Guard is a production-ready security library for FastAPI applications that provides:

- IP control and rate limiting
- Request logging and monitoring
- Penetration attempt detection
- Security headers management
- Redis-based distributed caching

- **PyPI Package**: `fastapi-guard`
- **Import Name**: `guard`
- **Python Support**: 3.10, 3.11, 3.12, 3.13, 3.14
- **Package Manager**: uv (modern Python package manager)
- **Build System**: Docker + Make

## Ecosystem Position

As of v5.0.0, fastapi-guard is a **thin adapter** over [guard-core](https://github.com/rennf93/guard-core). All security logic (models, handlers, decorators, detection engine, protocols, utilities) lives in the `guard_core` package. This repo contains only the FastAPI/Starlette integration layer.

```
guard-core (engine, PyPI dependency)   <- all security logic
└── fastapi-guard (this repo)          <- ASGI middleware adapter for FastAPI/Starlette
    ├── flaskapi-guard                 <- sibling adapter (Flask extension, sync mirror)
    ├── djapi-guard                    <- sibling adapter (Django middleware, sync mirror)
    └── tornadoapi-guard               <- sibling adapter (Tornado handler/middleware)
```

Because FastAPI/Starlette is async, this adapter imports `guard_core.*` directly (the sync adapter siblings import the unasync-generated `guard_core.sync.*` mirror instead).

### Package Components

- **`guard/__init__.py`** - Re-exports 20+ items from `guard_core` so users can `from guard import SecurityConfig, SecurityDecorator, ...` without knowing about guard-core. Also exports `SecurityMiddleware` from `guard.middleware`.
- **`guard/middleware.py`** - `SecurityMiddleware` extends Starlette's `BaseHTTPMiddleware`. It wraps Starlette `Request`/`Response` objects via adapters, delegates all security checks to guard-core's pipeline, and orchestrates initialization, event dispatch, metrics, and response processing using guard-core modules (`core.checks`, `core.events`, `core.initialization`, `core.responses`, `core.routing`, `core.validation`, `core.bypass`, `core.behavioral`).
- **`guard/adapters.py`** - Protocol adapters that bridge Starlette types to guard-core's framework-agnostic protocols: `StarletteGuardRequest` (adapts `starlette.requests.Request` to `GuardRequest`), `StarletteGuardResponse` (adapts `starlette.responses.Response` to `GuardResponse`), `StarletteResponseFactory` (creates response adapters), and the lifecycle helpers `wrap_call_next()` / `unwrap_response()`.
- **`guard/lifespan.py`** - `guard_lifespan`, `make_lifespan`, and `guard_startup` warm guard-core's shared-state registry at app startup so initialization does not happen on the first request.
- **`guard/_middleware_state.py`** - `MiddlewareState` registry keyed on the `SecurityConfig` instance and the resolved decorator handler, so two middleware instances sharing one config only share a pipeline when they resolve the same decorator handler.
- **`guard/_decorator_adoption.py`** - Resolves and adopts the app's registered `guard_decorator` (`resolve_app_state_decorator`, `adopt_app_state_decorator`) so the pipeline can be derived from the registered route config.
- **`guard/status.py`** - `add_status_route` helper exposing a guard status endpoint.
- **`guard/websocket.py`** - WebSocket support (`WebSocketCloseReason`, a `_WebSocketGuardRequest` adapter for WebSocket connections).

### Public API

All public imports go through `guard`:

```python
from guard.middleware import SecurityMiddleware
from guard import SecurityConfig, SecurityDecorator, RouteConfig
from guard import IPBanManager, RateLimitManager, RedisManager
from guard import GeoIPHandler, RedisHandlerProtocol
```

## Boundary Rules

- **This repo MUST NOT** contain security logic (checks, handlers, models, detection patterns). Those belong in [guard-core](https://github.com/rennf93/guard-core). A change that looks like a security fix belongs upstream in guard-core, not here.
- **This repo MUST** wire Starlette/FastAPI native types to guard-core's `GuardRequest`, `GuardResponse`, and `GuardResponseFactory` protocols via `guard/adapters.py`.
- **This repo MUST** keep `SecurityMiddleware` as a thin orchestrator that delegates to `SecurityCheckPipeline`; do not fork or reimplement pipeline behavior.
- **This repo MUST** re-export new guard-core public surface from `guard/__init__.py` when it becomes part of the adapter's user-facing API.
- This repo should only change when:
  - The Starlette/FastAPI adapter layer needs updates
  - New guard-core exports need to be re-exported from `guard/__init__.py`
  - FastAPI-specific middleware orchestration changes

## Quick Start

```bash
# Install dependencies with uv
make install-dev

# Run tests locally
make local-test

# Start example application
make start-example

# Run linting and formatting
make fix
```

## Development Commands

### Package Management (uv)

- `make install` - Install core dependencies
- `make install-dev` - Install with dev dependencies
- `make lock` - Update lock file
- `make upgrade` - Upgrade lock dependencies and install
- `uv sync` - Sync dependencies from lock file

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rennf93/fastapi-guard](https://github.com/rennf93/fastapi-guard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
