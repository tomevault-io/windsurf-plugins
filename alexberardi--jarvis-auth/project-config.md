---
trigger: always_on
description: The identity backbone. Three independent auth modes (user JWT, app-to-app credentials, node credentials), full household multi-tenancy, and an admin surface for credential lifecycle. Everything downstream of identity flows through here.
---

# jarvis-auth

The identity backbone. Three independent auth modes (user JWT, app-to-app credentials, node credentials), full household multi-tenancy, and an admin surface for credential lifecycle. Everything downstream of identity flows through here.

---

## What this service is (and isn't)

| Mode | Who | Header / Token | Where validated |
|---|---|---|---|
| **User JWT** | Humans (admin UI, web, mobile) | `Authorization: Bearer <jwt>` | **Locally** by each service using shared `AUTH_SECRET_KEY` — no round-trip to auth |
| **App-to-app** | Services calling each other | `X-Jarvis-App-Id` + `X-Jarvis-App-Key` | Round-trip to `/internal/validate-app` (or `/internal/app-ping`) |
| **Node** | Pi Zero nodes | `X-API-Key: node_id:node_key` (service-side header convention) | Round-trip to `/internal/validate-node` (which also checks per-service access) |

**Not** a:
- Permissions / RBAC engine for application data (only household-level role hierarchy: MEMBER < POWER_USER < ADMIN, used for household-scoped actions)
- Session store — JWTs are self-contained; refresh tokens are the only server-side session state
- Identity provider for third-party SSO (no OIDC/OAuth flows; local user/password only)

---

## Quick Reference

```bash
# Local dev (Docker default port 7701)
poetry install
cp env.template .env  # set AUTH_SECRET_KEY, DATABASE_URL, JARVIS_AUTH_ADMIN_TOKEN at minimum
alembic upgrade head
docker-compose -f docker-compose.dev.yaml up --build
# or local: poetry run uvicorn jarvis_auth.app.main:app --reload --port 7701

# Test
poetry run pytest
```

---

## Dependency graph

**Upstream (jarvis-auth depends on):**
- **PostgreSQL** (required) — users, refresh tokens, app clients, nodes, households, invites, settings
- **jarvis-logs** (optional, port 7702) — centralized logging; falls back to console on failure
- **jarvis-config-service** (optional, port 7700) — service discovery. Auth makes no outbound calls on the hot path, with **one exception**: `DELETE /auth/me` (account deletion) fans out a best-effort user-data purge to `jarvis-command-center` and `jarvis-notifications` (`DELETE /api/v0/me/data`, forwarding the user's Bearer token). Their URLs are resolved from config-service, or via `JARVIS_COMMAND_CENTER_URL` / `JARVIS_NOTIFICATIONS_URL` overrides. The purge is tolerant-blocking: unreachable services are skipped (best effort), but a downstream 5xx aborts the deletion (502) before any local data is touched.

**Downstream (depends on jarvis-auth):**
- **All services that validate JWTs** — share `AUTH_SECRET_KEY`, validate locally (no network call)
- **All services that need app-to-app auth** — round-trip to `/internal/app-ping` or `/internal/validate-app`
- **All services that accept nodes** (command-center, whisper, tts, logs) — round-trip to `/internal/validate-node`
- **jarvis-command-center** — also calls `/internal/validate-household-access`, `/internal/validate-node-household`, `/internal/users/batch` (speaker resolution)
- **jarvis-config-service** — uses `JARVIS_AUTH_ADMIN_TOKEN` to call `/admin/app-clients` during first-boot bootstrap
- **Browser clients**: `jarvis-admin`, `jarvis-web`, `jarvis-node-mobile` — hit `/auth/login` etc. directly (CORS allow-listed)

**Impact if down:**
- New user logins fail
- App-to-app authentication fails (anything calling `/internal/validate-app` returns 5xx) → cascade failure across the stack
- Node authentication fails → nodes lose access
- **Existing user JWTs keep working until expiry** (30min default) because validation is local-only

---

## Lifecycle / common operations

### 1. First-boot superuser bootstrap

```
GET  /auth/setup-status        → {"needs_setup": true} if no superusers
POST /auth/setup                → creates first user as superuser + admin of a new "My Home" household
```

The installer calls this once. After that, the endpoint refuses (409) — there's no other path to creating a superuser today (no env-var seeding). **Don't add one without a real reason** — restricting superuser creation to a one-shot bootstrap is a deliberate safety measure.

### 2. User register / login / refresh

`POST /auth/register` (or `/auth/login`) → returns `{access_token, refresh_token, user, household_id}`.

JWT claims always include: `sub` (user_id), `email`, `is_superuser`, `household_id` (current active household), `jti`, `iat`, `exp`. Services that need household scoping read `household_id` directly from the token.

**Token lifecycle:**
- Access token: 30 min (configurable via `ACCESS_TOKEN_EXPIRE_MINUTES`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexberardi/jarvis-auth](https://github.com/alexberardi/jarvis-auth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
