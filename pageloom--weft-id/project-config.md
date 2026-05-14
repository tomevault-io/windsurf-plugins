---
trigger: always_on
description: automates first-time setup (downloads files, generates secrets, prompts for domain/SMTP, writes `.env`).
---

# Project Instructions

## What This Project Is

WeftID is a multi-tenant identity federation platform that acts as middleware between applications and identity providers (Okta, Entra ID, Google Workspace, SAML/OIDC). Core capabilities:

- SAML 2.0 and OAuth2 identity provider integration
- SAML 2.0 Identity Provider for downstream service providers (SSO assertions, per-SP signing certificates)
- Multi-factor authentication (TOTP-based with backup codes)
- User lifecycle management with inactivation/reactivation workflows
- Comprehensive audit logging and activity tracking
- Tenant-isolated data with Row-Level Security (RLS)

## Before Starting Work

**Read `.claude/THOUGHT_ERRORS.md`** for common mistakes to avoid. Key gotchas:

- **Tests**: Use `make test` or `poetry run python -m pytest` (not `pytest` directly)
- **Code quality**: Run `make check` (lint, format, type check, compliance)
- **UUIDs**: Convert to string when comparing across boundaries
- **Background jobs**: Restart worker container, not app container
- **Mocking sessions**: Patch `starlette.requests.Request.session`, not client cookies

## Git Commits

- Keep them short and to the point
- The summary should be short (80 chars or less)
- The description should include a short definition of what problem was addressed
- The description should then explain, tersely, how it was done
- Do NOT include Claude attributions in commit messages

## Architecture Overview

This project follows a layered architecture:

```
Request → Router → Service → Database → PostgreSQL
```

- **Routers** (`app/routers/`): HTTP/template layer only. Never import database modules directly.
- **Services** (`app/services/`): Business logic and authorization. Receives `RequestingUser`, returns Pydantic schemas, raises `ServiceError` subclasses.
- **Database** (`app/database/`): SQL execution with tenant scoping. Returns dicts.

### Authentication vs Authorization

- **Authentication** (router layer): FastAPI dependencies in `app/dependencies.py` and `app/api_dependencies.py` identify the caller and return a user dict. They redirect unauthenticated users.
- **Authorization** (service layer): Functions in `app/services/auth.py` check role-based access. They receive a `RequestingUser` and raise `ForbiddenError` if the role is insufficient.

## Key Files

| File | Purpose |
|------|---------|
| `app/pages.py` | Authorization registry. All routes must be registered here |
| `app/constants/event_types.py` | Event type registry for audit logging |
| `app/schemas/common.py` | `RequestingUser` TypedDict and common schemas |
| `app/services/exceptions.py` | ServiceError subclasses (ForbiddenError, NotFoundError, ValidationError) |
| `app/services/event_log.py` | `log_event()` function for audit logging |
| `app/services/activity.py` | `track_activity()` for read operation tracking |
| `app/utils/crypto.py` | HKDF key derivation from `SECRET_KEY` (session, MFA, SAML, email) |
| `app/utils/email.py` | All outbound emails. Shared layout with inline styles, branded header/footer |
| `app/utils/email_branding.py` | Fetches tenant logo PNG + name for email headers |
| `app/dev/preview_emails.py` | Sends all 15 email types to MailDev for visual testing |
| `.claude/BACKLOG.md` | Product backlog (pending items) |
| `.claude/BACKLOG_ARCHIVE.md` | Completed backlog items with acceptance criteria |
| `.claude/ITERATION_*.md` | Active iteration plans managed by `/lead` (gitignored) |
| `.claude/ISSUES.md` | Active quality/security issues (goal: keep empty) |
| `.claude/ISSUES_ARCHIVE.md` | Resolved issues with fix details |
| `app/services/service_providers.py` | SP registration, SSO response building |
| `app/routers/saml_idp/` | SAML IdP admin, SSO, metadata (package) |
| `app/database/service_providers.py` | SP database queries |
| `app/database/sp_signing_certificates.py` | Per-SP signing certificate queries |
| `app/templates/icons/` | SVG icon files (19 Heroicons outline, viewable as images) |
| `static/js/utils.js` | Shared `WeftUtils` JS object (modals, sticky bars, clipboard, locale) |
| `static/js/cytoscape.min.js` | Cytoscape.js graph library (group graph views) |
| `app/version.py` | Runtime version via `importlib.metadata` (falls back to baked-in `VERSION` file) |
| `docs/VERSIONING.md` | Semver policy: patch/minor/major definitions, identity-specific rules |
| `CHANGELOG.md` | Release changelog (Keep a Changelog format) |
| `Dockerfile` | Production multi-stage build (GHCR images, no dev deps) |
| `app/Dockerfile` | Dev build (used by `dev/docker-compose.yml`) |
| `dev/docker-compose.yml` | Dev compose (nginx, app, worker, db, maildev, memcached) |
| `deploy/docker-compose.yml` | Self-hosting compose (Caddy + GHCR image, automatic HTTPS) |
| `deploy/Caddyfile` | Caddy reverse proxy config (on-demand TLS for tenant subdomains) |
| `deploy/.env.example` | Production environment template with generation instructions |
| `deploy/install.sh` | Self-hosting install script (downloads files, generates secrets, writes .env) |
| `.github/workflows/publish.yml` | GHCR publish workflow (triggers on `v*.*.*` tags) |
| `app/cli/provision_tenant.py` | CLI to provision a tenant and super admin (`python -m app.cli.provision_tenant`) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Pageloom/weft-id](https://github.com/Pageloom/weft-id) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
