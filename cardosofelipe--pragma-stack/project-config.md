---
trigger: always_on
description: AI coding assistant context for FastAPI + Next.js Full-Stack Template.
---

# AGENTS.md

AI coding assistant context for FastAPI + Next.js Full-Stack Template.

## Quick Start

```bash
# Backend (Python with uv)
cd backend
make install-dev              # Install dependencies
make test                     # Run tests
uv run uvicorn app.main:app --reload  # Start dev server

# Frontend (Node.js)
cd frontend
bun install                   # Install dependencies
bun run dev                   # Start dev server
bun run generate:api          # Generate API client from OpenAPI
bun run test:e2e              # Run E2E tests
```

**Access points:**
- Frontend: **http://localhost:3000**
- Backend API: **http://localhost:8000**
- API Docs: **http://localhost:8000/docs**

Default superuser (change in production):
- Email: `admin@example.com`
- Password: `admin123`

## Project Architecture

**Full-stack TypeScript/Python application:**

```
├── backend/                 # FastAPI backend
│   ├── app/
│   │   ├── api/            # API routes (auth, users, organizations, admin)
│   │   ├── core/           # Core functionality (auth, config, database)
│   │   ├── repositories/   # Repository pattern (database operations)
│   │   ├── models/         # SQLAlchemy ORM models
│   │   ├── schemas/        # Pydantic request/response schemas
│   │   ├── services/       # Business logic layer
│   │   └── utils/          # Utilities (security, device detection)
│   ├── tests/              # 96% coverage, 987 tests
│   └── alembic/            # Database migrations
│
└── frontend/               # Next.js 16 frontend
    ├── src/
    │   ├── app/           # App Router pages (Next.js 16)
    │   ├── components/    # React components
    │   ├── lib/
    │   │   ├── api/       # Auto-generated API client
    │   │   └── stores/    # Zustand state management
    │   └── hooks/         # Custom React hooks
    └── e2e/               # Playwright E2E tests (56 passing)
```

## Critical Implementation Notes

### Authentication Flow
- **JWT-based**: Access tokens (15 min) + refresh tokens (7 days)
- **OAuth/Social Login**: Google and GitHub with PKCE support
- **Session tracking**: Database-backed with device info, IP, user agent
- **Token refresh**: Validates JTI in database, not just JWT signature
- **Authorization**: FastAPI dependencies in `api/dependencies/auth.py`
  - `get_current_user`: Requires valid access token
  - `get_current_active_user`: Requires active account
  - `get_optional_current_user`: Accepts authenticated or anonymous
  - `get_current_superuser`: Requires superuser flag

### OAuth Provider Mode (MCP Integration)
Full OAuth 2.0 Authorization Server for MCP (Model Context Protocol) clients:
- **Authorization Code Flow with PKCE**: RFC 7636 compliant
- **JWT access tokens**: Self-contained, no DB lookup required
- **Opaque refresh tokens**: Stored hashed in database, supports rotation
- **Token introspection**: RFC 7662 compliant endpoint
- **Token revocation**: RFC 7009 compliant endpoint
- **Server metadata**: RFC 8414 compliant discovery endpoint
- **Consent management**: User can review and revoke app permissions

**API endpoints:**
- `GET /.well-known/oauth-authorization-server` - Server metadata
- `GET /oauth/provider/authorize` - Authorization endpoint
- `POST /oauth/provider/authorize/consent` - Consent submission
- `POST /oauth/provider/token` - Token endpoint
- `POST /oauth/provider/revoke` - Token revocation
- `POST /oauth/provider/introspect` - Token introspection
- Client management endpoints (admin only)

**Scopes supported:** `openid`, `profile`, `email`, `read:users`, `write:users`, `admin`

**OAuth Configuration (backend `.env`):**
```bash
# OAuth Social Login (as OAuth Consumer)
OAUTH_ENABLED=true                           # Enable OAuth social login
OAUTH_AUTO_LINK_BY_EMAIL=true                # Auto-link accounts by email
OAUTH_STATE_EXPIRE_MINUTES=10                # CSRF state expiration

# Google OAuth
OAUTH_GOOGLE_CLIENT_ID=your-google-client-id
OAUTH_GOOGLE_CLIENT_SECRET=your-google-client-secret

# GitHub OAuth
OAUTH_GITHUB_CLIENT_ID=your-github-client-id
OAUTH_GITHUB_CLIENT_SECRET=your-github-client-secret

# OAuth Provider Mode (as Authorization Server for MCP)
OAUTH_PROVIDER_ENABLED=true                  # Enable OAuth provider mode
OAUTH_ISSUER=https://api.yourdomain.com      # JWT issuer URL (must be HTTPS in production)
```

### Database Pattern
- **Async SQLAlchemy 2.0** with PostgreSQL
- **Connection pooling**: 20 base connections, 50 max overflow
- **Repository base class**: `repositories/base.py` with common operations
- **Migrations**: Alembic with helper script `migrate.py`
  - `python migrate.py auto "message"` - Generate and apply
  - `python migrate.py list` - View history

### Frontend State Management
- **Zustand stores**: Lightweight state management
- **TanStack Query**: API data fetching/caching
- **Auto-generated client**: From OpenAPI spec via `bun run generate:api`
- **Dependency Injection**: ALWAYS use `useAuth()` from `AuthContext`, NEVER import `useAuthStore` directly

### Internationalization (i18n)
- **next-intl v4**: Type-safe internationalization library
- **Locale routing**: `/en/*`, `/it/*` (English and Italian supported)
- **Translation files**: `frontend/messages/en.json`, `frontend/messages/it.json`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cardosofelipe/pragma-stack](https://github.com/cardosofelipe/pragma-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
