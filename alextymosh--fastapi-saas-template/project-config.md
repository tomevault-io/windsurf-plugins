---
trigger: always_on
description: Execution contract for AI coding agents. Enforce architecture, constraints, and patterns. Prefer consistency over creativity.
---

# AGENTS.md

## Purpose

Execution contract for AI coding agents. Enforce architecture, constraints, and patterns. Prefer consistency over creativity.

## Instruction Priority

1. User task.
2. This file.
3. Existing repository patterns.
4. README.
5. Framework defaults.

## Before starting work

Codex/AI agents must:

1. Confirm current working directory and branch.
2. Read `AGENTS.md`.
3. Read `SESSION_NOTES.md` if present.
4. Read relevant docs from `backend/docs`.
5. Run `git status`.
6. Report the plan before editing.

## Structure

The backend is a modular monolith with explicit layers:

- `api`
- `services`
- `repositories`
- `schemas`
- `models`
- `core`

Domain modules live under:

```text
backend/app/<domain>/
  api/
  services/
  repositories/
  schemas/
  models/
```

Router registration entry point:

```text
backend/app/api/master_router.py
```

Expected flow:

`HTTP -> API -> Service -> Repository -> Service -> Response`

## Router contract

- Domain router files live in `backend/app/<domain>/api/*.py`.
- One file defines one router.
- Each router must define tags.
- Route paths must be defined inside domain router files.
- Routers should be attached in deterministic order with numbered comments when practical.
- `backend/app/api/master_router.py` is the only domain-router registration point.
- All API routes are attached to `v1_router`.
- Version prefix `/api/v1` is applied centrally in `master_router.py` through settings.
- `main.py` must not register domain routers directly.
- Do not pass extra prefixes in `include_router`, except the central version prefix.

## Layer responsibilities

- API handlers must stay thin.
- API must not contain business logic.
- Services contain business logic, orchestration, and authorization decisions.
- Repositories handle database access only.
- API layer must not access the database directly.
- Do not use raw SQL outside repositories unless explicitly justified.
- Use FastAPI dependency injection with `Depends`; avoid hidden globals.
- Use async only for database/external I/O; pure CPU logic should be sync.

## Persistence ownership

- Domain repositories own persistence for their aggregate tables.
- Platform services may orchestrate privileged workflows by calling domain repositories and platform-owned repositories, but must not duplicate basic persistence access for domain-owned tables.
- Platform repositories are allowed only for platform-owned tables, such as `platform_staff`, or dedicated platform read models/reporting queries that intentionally span multiple aggregates.
- Ownership mapping:
  - `users` table -> `UserRepository`
  - `organisations` table -> `OrganisationRepository`
  - `memberships` table -> `MembershipRepository`
  - `platform_staff` table -> `PlatformStaffRepository`
- Platform services own orchestration, permissions, audit event creation, conflict/not-found mapping, and state-transition decisions; they must not build SQLAlchemy queries for domain-owned aggregate tables.
- Platform organisation visibility is explicit: platform admin endpoints may include soft-deleted organisations for operational, audit, support, compliance, or recovery workflows; tenant-facing organisation endpoints must exclude soft-deleted organisations by default. `OrganisationRepository.get_by_id(..., include_deleted=False)` is the safe default, and platform services must pass `include_deleted=True` when they intentionally need deleted-organisation visibility.

## Transaction ownership

- Repositories may use `flush()` and `refresh()`, but must not call `commit()` or `rollback()`.
- Application services should not commit by default. Services orchestrate business rules, repository calls, and audit writes inside a transaction provided by the caller.
- Write API dependencies own transaction boundaries using `async with session.begin()` after authentication and rate limiting have completed.
- Read endpoints should use the lazy request-scoped session and should not open explicit transactions unless consistency requirements justify it.
- CLI commands and background workers must create their own explicit transaction boundary.
- Do not add global transaction middleware. It can start database work too early, weaken early auth/rate-limit short-circuiting, and make transaction scope less visible.

## API response contract

- Single resource: clean REST response.
- Collections: `{ "data": [], "meta": {}, "links": {} }`.
- Errors: Problem Details style with `application/problem+json`.
- Operational endpoints may return endpoint-specific payloads.

## Error handling contract

- API layer must not format business errors manually.
- API layer must not use `try`/`except` for business-flow errors.
- Services raise application/domain exceptions.
- Global FastAPI handlers format Problem Details responses.
- Do not leak internals, stack traces, tokens, secrets, or raw sensitive data.
- Token/credential-like flows must not reveal whether a token exists, expired, was revoked, or was already used; normalise external error responses.

## Security and auth

- JWT authentication uses Keycloak as identity provider.
- Authentication and authorization are separate concerns.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AlexTymosh/fastapi-saas-template](https://github.com/AlexTymosh/fastapi-saas-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
