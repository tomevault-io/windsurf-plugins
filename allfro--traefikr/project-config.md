---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Traefikr is a Traefik configuration management API that provides a REST API for managing Traefik v3.6 resources (routers, services, middlewares, etc.) with comprehensive JSON schema validation. The backend is written in Go and uses SQLite for persistence.

## Build and Development Commands

### Docker Development
```bash
# Build and start all services (Traefik + Backend)
docker-compose up --build

# Rebuild backend only
docker-compose build backend

# Stop and remove containers
docker-compose down

# Remove backend data volume
docker volume rm traefikr_backend-data
```

### Backend Development
```bash
cd backend

# Install dependencies
go mod download

# Build binary
go build -o traefikr .

# Build static binary for production (used in Dockerfile)
CGO_ENABLED=1 GOOS=linux go build \
    -a \
    -ldflags '-linkmode external -extldflags "-static" -s -w' \
    -o main .

# Run locally
TRAEFIKR_DB_PATH=./traefikr.db TRAEFIKR_PORT=8080 TRAEFIK_API_URL=http://localhost:8080 ./traefikr
```

### Testing
```bash
# Test authentication flows
./tests/test_auth.sh

# Create test resources
./tests/create_fixed_resources.sh

# Create all middleware types
./tests/create_middlewares.sh

# Delete all resources
./tests/delete_all_resources.sh
```

## Architecture

### Dual Authentication System

Traefikr implements **two separate authentication mechanisms** - this is critical to understand:

1. **JWT Authentication** (User/Admin access)
   - Used for all CRUD operations on resources
   - Login at `/api/auth/login` with username/password returns JWT token
   - Token must be passed as `Authorization: Bearer <token>` header
   - Tokens expire after 24 hours
   - Implemented in `middleware/jwt.go`

2. **API Key Authentication** (Traefik polling)
   - **ONLY** used for `/api/config` endpoint (Traefik HTTP provider polling)
   - API keys are standalone, not linked to users
   - Created via `/api/http/provider` endpoint (requires JWT)
   - Passed as `x-traefikr-key` header
   - Implements conditional authentication: public if no API keys exist (bootstrap mode), requires auth once keys are created
   - Implemented in `middleware/auth.go`

**IMPORTANT**: These two authentication systems must remain separate. Do not mix them or allow x-traefikr-key to be used for CRUD operations.

### Route Structure

```
Public Endpoints:
- GET  /health                              # Health check
- POST /api/auth/login                      # Get JWT token
- GET  /api/{protocol}/{type}/schema.json   # Get validation schema

Conditional Auth (API Key for Traefik):
- GET  /api/config                          # Traefik HTTP provider endpoint

Protected Endpoints (JWT Required):
- GET    /api/{protocol}/{type}                    # List resources (supports ?traefik=true|false)
- GET    /api/{protocol}/{type}/{nameProvider}     # Get resource
- POST   /api/{protocol}/{type}                    # Create resource
- PUT    /api/{protocol}/{type}/{nameProvider}     # Update resource
- DELETE /api/{protocol}/{type}/{nameProvider}     # Delete resource
- GET    /api/entrypoints                          # List entrypoints
- GET    /api/entrypoints/{name}                   # Get entrypoint
- GET    /api/http/provider                        # List API keys
- POST   /api/http/provider                        # Create API key
- DELETE /api/http/provider/{id}                   # Delete API key
```

### Query Parameters

The List Resources endpoint (`GET /api/{protocol}/{type}`) supports an optional `traefik` query parameter:

- **traefik=false** (default): Returns only resources from the local database
- **traefik=true**: Returns resources from both database and Traefik API (merged)

Examples:
```bash
# Get only database resources (default)
curl -H "Authorization: Bearer $JWT" http://localhost:8000/api/http/routers

# Explicitly request only database resources
curl -H "Authorization: Bearer $JWT" http://localhost:8000/api/http/routers?traefik=false

# Get merged resources from database and Traefik
curl -H "Authorization: Bearer $JWT" http://localhost:8000/api/http/routers?traefik=true
```

### Protocol and Resource Types

Supported protocols: `http`, `tcp`, `udp`

Resource types by protocol:
- **http**: routers, services, middlewares, serversTransport, tls
- **tcp**: routers, services, middlewares, serversTransport, tls
- **udp**: routers, services, middlewares

### Resource Naming Convention

Resources are identified by `name@provider` format (e.g., `my-router@http`). When creating resources:
- Request body contains `name` field without `@provider` suffix
- Response includes full `name@provider` identifier
- Database stores name and provider separately

### Data Flow

1. **Resource Creation/Update**:
   - User authenticates with JWT
   - Request validated against JSON schema (in `schemas/`)
   - Resource stored in SQLite database
   - Traefik polls `/api/config` to get updated configuration

2. **Resource Listing**:
   - Fetches resources from both Traefik API (live) and local database
   - Merges results, prioritizing database for enabled/disabled state

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [allfro/traefikr](https://github.com/allfro/traefikr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
