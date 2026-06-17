---
trigger: always_on
description: **Generated:** 2026-03-31 21:38:42 CST
---

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-03-31 21:38:42 CST  
**Commit:** 3afb500  
**Branch:** main

## Scope
- Applies repo-wide.
- Nearest child `AGENTS.md` overrides local work.
- Read next: `src/AGENTS.md`, `src/routes/AGENTS.md`, `frontend/AGENTS.md`, `frontend/app/AGENTS.md`, `frontend/lib/AGENTS.md`.

## Overview
TmpMail is a Rust Axum API plus a Next.js App Router frontend. Persistence is PostgreSQL-only; startup accepts either an explicit `TMPMAIL_DATABASE_URL` or the bundled compose `TMPMAIL_POSTGRES_*` settings, then applies `migrations/` automatically.

## Structure
```text
.
├── src/                 # Rust backend core, workers, auth, storage, routes
├── migrations/          # PostgreSQL schema
├── frontend/            # Next.js app, proxies, UI, i18n
├── scripts/             # Preferred local dev + smoke helpers
├── Dockerfile           # API image
├── frontend/Dockerfile  # Frontend image
├── compose.yaml         # Default orchestration for API + frontend + postgres + inbucket
├── .env.example         # Canonical env template
├── README.md            # Operational behavior and deployment notes
└── data/                # Runtime state only; ignored
```

## Where to look
| Task | Location | Notes |
|---|---|---|
| Bring the stack up/down | `compose.yaml`, `scripts/dev-up.sh`, `scripts/dev-down.sh` | Default workflow is plain `docker compose`; `dev-up` can auto-create `.env`, ask for script language, prompt for required values plus proxy mode, hard-fail when host `25/TCP` is not both published to Inbucket and reachable on `127.0.0.1:25`, wait for `api` and `frontend` readiness, print DNS follow-up, warn-but-continue when `TMPMAIL_MAIL_EXCHANGE_HOST` is not resolvable yet, hard-fail only when it resolves but `:25` cannot connect, and then optionally prompt to clean TmpMail-owned dangling Docker images |
| Backend startup | `src/main.rs`, `src/app.rs` | Config, worker spawning, router assembly |
| HTTP route map | `src/routes/mod.rs` | `api_router()` vs `stream_router()` |
| Storage backend | `src/app_store.rs`, `src/pg_store.rs` | PostgreSQL-only business storage |
| Runtime overrides | `src/config.rs`, `src/admin_state.rs` | Env parsing plus persisted admin overrides |
| Frontend shell | `frontend/app/[locale]/layout.tsx`, `frontend/app/[locale]/page.tsx` | Locale shell, providers, inbox/guest flow |
| Frontend proxy edges | `frontend/app/api/mail/route.ts`, `frontend/app/api/sse/route.ts` | REST proxy vs SSE proxy |
| Configurable admin path | `frontend/proxy.ts`, `frontend/lib/admin-entry.ts` | Never assume `/admin` is fixed |
| Frontend transport/helpers | `frontend/lib/api.ts` | Central fetch, errors, session-adjacent helpers |
| Minimal end-to-end validation | `scripts/smoke.sh` | Health, auth, inbox, raw message, frontend `/en` |

## Code map
| Symbol | Type | Location | Refs | Role |
|---|---|---|---:|---|
| `build_router` | fn | `src/app.rs` | 6 | Merges protected API routes with the separate stream router |
| `api_router` | fn | `src/routes/mod.rs` | 2 | Canonical REST route table |
| `AppStore` | struct | `src/app_store.rs` | 67 | PostgreSQL facade used across routes and workers |
| `AuthProvider` | component | `frontend/contexts/auth-context.tsx` | 4 | Frontend auth state boundary mounted from locale layout |

## Shared verification
- Backend changes: `cargo test` and `cargo build --release`
- Frontend changes: `cd frontend && npm run lint`
- Frontend route/env/proxy changes: also run `cd frontend && npm run build`
- Repo-level sanity: `./scripts/smoke.sh`
- Storage or schema changes: verify happy path plus migration/restore behavior

## Conventions
- Prefer plain `docker compose up -d --build` / `docker compose down` for the default deployment path; `scripts/dev-up.sh` and `scripts/dev-down.sh` are optional local helpers, and `dev-up` can auto-create `.env` from `.env.example`, ask for script language with Simplified Chinese as the default, prompt for missing required deployment values plus `TMPMAIL_TRUST_PROXY_HEADERS` deployment mode, validate that host `25/TCP` really publishes to Inbucket SMTP and is reachable on `127.0.0.1:25`, wait for `api` / `frontend` readiness, print a single mail-host A/AAAA DNS step after startup, warn-but-continue when `TMPMAIL_MAIL_EXCHANGE_HOST` is not resolvable yet, hard-fail only when it resolves but `:25` cannot connect, and then optionally prompt to clean TmpMail-owned dangling Docker images without touching other projects' caches.
- `.env.example` is the source of truth for new settings. Update `README.md` when admin, JWT, transport, or deployment knobs change.
- Env namespaces are deliberate: `TMPMAIL_*` backend/runtime plus Docker build overrides, `NEXT_PUBLIC_TMPMAIL_*` browser-visible frontend, and `INBUCKET_*` compose-internal container envs.
- Security-sensitive envs now include `TMPMAIL_JWT_SECRET`, `TMPMAIL_ALLOW_INSECURE_DEV_SECRETS`, `TMPMAIL_ADMIN_PASSWORD`, `TMPMAIL_ADMIN_PASSWORD_MODE`, `TMPMAIL_DATABASE_URL`, `TMPMAIL_POSTGRES_PASSWORD`, `TMPMAIL_TRUST_PROXY_HEADERS`, `TMPMAIL_CONTAINER_UID`, `TMPMAIL_CONTAINER_GID`, and `TMPMAIL_POSTGRES_BIND_IP`; keep docs and compose defaults aligned when they change.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ign1x/tmpmail](https://github.com/Ign1x/tmpmail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
