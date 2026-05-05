---
trigger: always_on
description: Validates:
---

# Wildbox Security Platform - AI Agent Instructions

**Version:** 1.0 | **Last Updated:** October 2025

## 🏗️ Architecture Overview

Wildbox is a microservices-based security operations platform with **11 containerized services** orchestrated through Docker Compose. The gateway acts as the intelligent entry point, routing all requests through OpenResty/Nginx with Lua-based authentication and rate limiting.

### Service Communication Pattern

```
Browser/API Client → Gateway (port 80/443) → Backend Services
                         ↓
                   Identity Service (8001) ← Authentication/Authorization
```

**Critical**: All production traffic MUST flow through the gateway. Direct service access (ports 8000-8019) is for development only.

### Core Services & Ports

| Service | Port | Purpose | Auth Method |
|---------|------|---------|-------------|
| **gateway** | 80/443 | OpenResty API gateway with Lua auth | JWT + API Key |
| **identity** | 8001 | FastAPI auth service (JWT, teams, subscriptions) | JWT internally |
| **api** (tools) | 8000 | FastAPI security tools (55+ tools) | API Key |
| **data** | 8002 | Django threat intelligence & IOCs | API Key |
| **guardian** | 8013 | Django vulnerability management | API Key |
| **responder** | 8018 | FastAPI incident response & playbooks | API Key |
| **agents** | 8006 | FastAPI AI-powered analysis (GPT-4o) | API Key |
| **cspm** | 8019 | FastAPI cloud security (200+ checks) | API Key |
| **sensor** | 8004 | Rust endpoint monitoring (osquery) | Certificate |
| **dashboard** | 3000 | Next.js 14 frontend (App Router) | Session + JWT |
| **automations** | 5678 | n8n workflow automation | Basic Auth |

### Shared Infrastructure

- **postgres**: Single PostgreSQL 15 instance with separate databases (`identity`, `data`, `guardian`, etc.)
- **wildbox-redis**: Single Redis 7 instance with logical database separation (DB 0-15)
  - DB 0: Identity cache
  - DB 1: Guardian cache
  - DB 5: Gateway auth cache
  - etc.

## 🔐 Authentication Architecture

### Gateway-Based Authentication Flow

1. **Request arrives** at gateway with Bearer token or API key
2. **Gateway Lua script** (`/nginx/lua/auth_handler.lua`) extracts token
3. **Identity service validates** via internal `/internal/authorize` endpoint
4. **Gateway injects headers** to backend: `X-Wildbox-User-ID`, `X-Wildbox-Team-ID`, `X-Wildbox-Plan`, `X-Wildbox-Role`
5. **Backend services trust** these headers (never exposed externally)

### API Key Format

```
wsk_<4-char-prefix>.<64-char-hex>
Example: wsk_a3f4.e7d2c8b1...
```

- Generated in identity service via `generate_api_key()` in `app/auth.py`
- Stored as SHA256 hash in database
- Team-scoped with plan-based permissions

### Frontend API Client Pattern

**Dashboard uses gateway-aware clients** (see `src/lib/api-client.ts`):

```typescript
// CORRECT: Uses gateway when NEXT_PUBLIC_USE_GATEWAY=true
const response = await identityClient.get('/auth/me')
// Transforms to: http://gateway/api/v1/identity/auth/me

// INCORRECT: Direct service access in production
const response = await fetch('http://localhost:8001/api/v1/auth/me')
```

**Path transformation rules:**
- Identity: `/api/v1/auth/*` → Gateway: `/auth/*`
- Data: `/api/v1/data/*` → Gateway: `/api/v1/data/*`
- Guardian: `/api/v1/vulnerabilities/*` → Gateway: `/api/v1/guardian/*`

## 🚀 Developer Workflows

### Starting the Platform

```bash
# Full stack (recommended)
docker-compose up -d

# Wait for initialization (critical for first run)
sleep 180

# Verify health
./comprehensive_health_check.sh
```

**First-time setup creates**:
- Default admin user: `admin@wildbox.security` / `CHANGE-THIS-PASSWORD`
- API keys for inter-service communication
- Database schemas via migrations

### Debugging Services

```bash
# View logs for specific service
docker-compose logs -f [service-name]

# Check authentication flow
docker-compose logs -f gateway | grep "auth_handler"

# Monitor gateway routing decisions
docker-compose logs -f gateway | grep "proxy_pass"

# Test service directly (bypassing gateway)
curl http://localhost:8001/health
```

### Common Issues & Fixes

**"Gateway upstream host not found"**: Services starting in wrong order
```bash
docker-compose restart gateway
```

**"Browser cache showing old data"**: Frontend caching issue
```bash
# Clear browser cache or use incognito mode
```

**"Database does not exist"**: Migration not run
```bash
docker-compose exec postgres createdb -U postgres [db-name]
docker-compose restart [service-name]
```

## 📦 Adding New Features

### Creating a New API Endpoint

1. **Backend** (FastAPI example in `identity`):
   ```python
   # app/api_v1/endpoints/new_feature.py
   @router.get("/new-endpoint")
   async def new_endpoint(
       current_user: User = Depends(current_active_user)
   ):
       # Gateway already validated auth
       # Trust X-Wildbox-* headers from gateway
       return {"data": "response"}
   ```

2. **Gateway routing** (add to `nginx/conf.d/wildbox_gateway.conf`):
   ```nginx
   location /api/v1/new-feature/ {
       access_by_lua_block {
           local auth_handler = require "auth_handler"
           auth_handler.authenticate()
       }
       proxy_pass http://identity_service;
   }
   ```

3. **Frontend client** (update `src/lib/api-client.ts`):
   ```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fabriziosalmi/wildbox](https://github.com/fabriziosalmi/wildbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
