---
trigger: always_on
description: FastAPI-based HTTP interface providing RESTful endpoints for SDK access and frontend communication, with unified Auth0/API key authentication.
---

# Airweave API Layer Rules

## Overview
FastAPI-based HTTP interface providing RESTful endpoints for SDK access and frontend communication, with unified Auth0/API key authentication.

## Architecture

### Structure
```
api/
├── v1/
│   ├── api.py              # Main router aggregation
│   └── endpoints/          # Individual endpoint modules
├── deps.py                 # Dependency injection & auth resolution
├── router.py              # Custom TrailingSlashRouter
├── middleware.py          # Request processing & CORS
└── auth.py               # Auth0 integration & token validation
```

### Endpoint Categories
- **Public API (SDK)**: `/sources/`, `/collections/`, `/source-connections/`
- **Internal Frontend**: `/users/`, `/organizations/`, `/api-keys/`, `/sync/`, `/dag/`, `/entities/`, `/destinations/`
- **Connect Frontend API**: `/connect/` - Short-lived session tokens for embedded frontend integration flows (Plaid-style Connect modal)

### Key Endpoints

#### Sources API
- **GET /sources/{short_name}**: Retrieves source details including:
  - Core metadata (name, description)
  - Authentication methods (`auth_methods`)
  - Configuration schemas (`auth_fields`, `config_fields`)
  - **Auth Provider Support** (`supported_auth_providers`): List of auth provider short names that support this source

#### Source Connections API
- **POST /source-connections**: Creates a new connection with validation:
  - For direct auth: validates credential format
  - For OAuth: handles authorization flows
  - For auth providers: validates provider exists and supports the source
- **POST /source-connections/{id}/verify-oauth**: Verifies claim-token ownership and triggers deferred sync. Called after the OAuth callback completes to prove the caller that initiated the flow is the one completing it. Required for all browser-based OAuth flows.

## Core Components

### 1. TrailingSlashRouter
```python
from airweave.api.router import TrailingSlashRouter
router = TrailingSlashRouter()  # Handles /endpoint and /endpoint/
```

### 2. API Context (deps.py)
The `get_context` dependency provides unified authentication and request context handling:

```python
@router.get("/")
async def my_endpoint(
    ctx: ApiContext = Depends(deps.get_context),
):
    # Provides: organization, user, logger, request_id, auth_method, analytics
```

`ApiContext` inherits from `BaseContext` (`core.context`), which provides organization identity and logger. `ApiContext` adds HTTP-specific fields: `request_id`, `auth_method`, `auth_metadata`, `analytics`.

CRUD operations accept `BaseContext` (the parent type), so `ApiContext` works everywhere. Background jobs and Temporal activities use `BaseContext` directly without API-specific fields.

**ApiContext Resolution Flow:**
1. Check auth method: system (dev), Auth0, or API key
2. Resolve organization ID from header or defaults
3. Validate organization access
4. Create contextual logger with request metadata
5. Return `ApiContext` (extends `BaseContext`) with injected logger and analytics

**Key Features:**
- Supports multiple auth methods simultaneously
- Organization context via `X-Organization-ID` header
- Automatic access validation
- Pre-configured contextual logger injected via dependency injection
- Request tracking with unique request IDs

### 3. Authentication Methods

#### Auth0 Integration (auth.py)
- Production: Real Auth0 with JWT validation
- Development: Mock Auth0 when `AUTH_ENABLED=false`
- Token verification: `get_user_from_token()` for WebSocket/SSE

#### API Key Authentication
- Header: `X-API-Key: <key>`
- Single organization scope
- Expiration validation
- No user context (service-to-service)

#### System Authentication
- Local development with `AUTH_ENABLED=false`
- Uses `FIRST_SUPERUSER` as default user
- Full access to all organizations

#### Connect Session Authentication
- Header: `Authorization: Bearer <session_token>` (10-minute TTL)
- Created via `POST /connect/sessions` with API key auth
- Scope: Single collection + optional integration restrictions
- Modes: `all`, `connect`, `manage`, `reauth` - control permitted operations

### 4. Middleware Stack (middleware.py)

**Request Processing Pipeline:**
1. `add_request_id`: Generates unique request ID for tracing
2. `log_requests`: Logs request details and duration
3. `exception_logging_middleware`: Catches and logs unhandled exceptions

**CORS Handling:**
- Dynamic origin validation
- Special handling for OAuth endpoints
- White-label endpoint support
- Credentials support for cross-origin requests

**Exception Handlers:**
- `validation_exception_handler`: Enhanced 422 errors with schema context
- `permission_exception_handler`: 403 for access violations
- `not_found_exception_handler`: 404 for missing resources

### 5. Context Cache Service (core/context_cache_service.py)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [airweave-ai/airweave](https://github.com/airweave-ai/airweave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
