---
trigger: always_on
description: Self-hosted Enterprise Architecture Management platform that creates a **digital twin of a company's IT landscape**. Fully admin-configurable metamodel — card types, fields, subtypes, relations, stakeholder roles, and calculated fields are all data, not code.
---

# Turbo EA

Self-hosted Enterprise Architecture Management platform that creates a **digital twin of a company's IT landscape**. Fully admin-configurable metamodel — card types, fields, subtypes, relations, stakeholder roles, and calculated fields are all data, not code.

**Current version**: see `/VERSION` (single source of truth for backend + frontend).

## Quick Start

```bash
cp .env.example .env          # Edit secrets and DB credentials
docker compose up --build -d  # Starts backend (port 8000) + frontend (port 8920)
```

The first user to register automatically gets the `admin` role. Set `SEED_DEMO=true` to pre-populate with the NexaTech Industries demo dataset. Add `--profile ai` to include the bundled Ollama container for AI description suggestions.

---

## AI Assistant Guidelines

When working on this codebase, follow these conventions:

### General Principles
- **Data-driven metamodel**: Card types, fields, subtypes, relations, stakeholder roles, and calculated fields are all stored as data (JSONB) in the database, not hardcoded. Never add new card types or fields in code — add them via the seed or admin API.
- **Cards, not fact sheets**: The codebase was fully renamed from "fact sheets" to "cards". Never introduce the old terminology.
- **Permission checks are mandatory**: Every mutating endpoint must call `PermissionService.require_permission()` or use the `require_permission()` dependency. Never bypass permission checks.
- **No raw SQL**: Use SQLAlchemy ORM for all database queries. Alembic for all schema changes.

### Backend Conventions
- All route handlers live in `backend/app/api/v1/`, one file per resource domain.
- New routes must be registered in `backend/app/api/v1/router.py`.
- Use `async def` for all route handlers and database operations.
- Permission checking: use `PermissionService.require_permission(db, user, "permission.key")` or the `require_permission("permission.key")` FastAPI dependency from `deps.py`.
- New permission keys must be added to `backend/app/core/permissions.py` (the single source of truth for all valid permission keys).
- New models go in `backend/app/models/` (one file per table) and must be imported in `backend/app/models/__init__.py`.
- Schema changes require a new Alembic migration in `backend/alembic/versions/` with sequential numbering (e.g., `036_description.py`).
- Sensitive values (SSO secrets, SMTP passwords) must use `encrypt_value()`/`decrypt_value()` from `backend/app/core/encryption.py`.
- Ruff linting: line length 100, rules E/F/I/N/W. Run `ruff check .` and `ruff format .`.
- **Before every commit**, run `cd backend && ruff format . && ruff check .` to ensure CI won't fail on formatting or lint errors. This is mandatory — do not skip it.

### MCP Server Conventions
- The MCP server lives in `mcp-server/` — a separate Python package (`turbo-ea-mcp`) with its own `pyproject.toml` and Dockerfile.
- It provides **read-only** AI tool access to EA data via the [Model Context Protocol](https://modelcontextprotocol.io/) (FastMCP library).
- **Two transport modes**: HTTP/SSE (production, via Docker `--profile mcp`) and stdio (local testing with Claude Desktop).
- **Authentication**: In HTTP mode, users authenticate via OAuth 2.1 delegated to the Turbo EA SSO provider. The MCP server resolves OAuth tokens to Turbo EA JWTs. In stdio mode, `TURBO_EA_EMAIL`/`TURBO_EA_PASSWORD` env vars are used for direct login.
- **Tools are read-only**: `search_cards`, `get_card`, `get_card_relations`, `get_card_hierarchy`, `list_card_types`, `get_relation_types`, `get_dashboard`, `get_landscape`. Do not add mutating tools without careful security review.
- **All data access respects RBAC**: The user's JWT is passed through to the backend API, so permission checks are enforced server-side.
- **Config** is in `mcp-server/turbo_ea_mcp/config.py` — reads from env vars (`TURBO_EA_URL`, `TURBO_EA_PUBLIC_URL`, `MCP_PUBLIC_URL`, `MCP_PORT`).
- **Tests** live in `mcp-server/tests/` and use `pytest` + `pytest-asyncio`. Run with `cd mcp-server && pip install -e ".[dev]" && pytest`.
- The MCP server shares the `/VERSION` file with backend/frontend for version consistency.

### Frontend Conventions
- Route-level pages use `lazy()` imports in `App.tsx` for code splitting.
- Shared hooks in `src/hooks/`, shared components in `src/components/`.
- Feature-specific components go in `src/features/{feature}/`.
- Use `api.get()`, `api.post()`, `api.patch()`, `api.delete()` from `src/api/client.ts` for all API calls.
- JWT token is in `sessionStorage` (not localStorage). Use `setToken()`/`clearToken()` from `client.ts`.
- All TypeScript interfaces live in `src/types/index.ts`.
- Use MUI 6 components — do not introduce other UI libraries.
- Icons use Google Material Symbols via the `MaterialSymbol` component.
- When nesting MUI Dialogs, use `disableRestoreFocus` on inner dialogs.

### Internationalization (i18n)
- **All user-facing strings must use translation keys**, never hardcoded English text. Use `useTranslation()` from `react-i18next` with the appropriate namespace.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vincentmakes/turbo-ea](https://github.com/vincentmakes/turbo-ea) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
