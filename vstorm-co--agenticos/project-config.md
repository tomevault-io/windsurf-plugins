---
trigger: always_on
description: Project-wide decisions and pointers to task-specific guidance. Read relevant rules,
---

# CLAUDE.md — AgenticOS

Project-wide decisions and pointers to task-specific guidance. Read relevant rules,
skills and documentation as needed. Keep detailed procedures and incident history
outside this file. Current user instructions and higher-priority session rules
take precedence.

## What this is

A self-hosted, open-source, multi-tenant platform for a company's AI agents.

**An agent is a versioned spec.** Users configure instructions, models, capabilities
and budgets in the UI, publish a version and can export it as YAML. The same agent
runs through web chat, the HTTP API and channel integrations. Change an individual
agent through its spec; change Python when implementing platform behaviour or
capabilities. Tools reach models through the capability registry.

**Stack:** FastAPI, Pydantic v2, Pydantic AI, PostgreSQL with pgvector, Redis,
Prefect, Next.js, React, bun, next-intl and MkDocs Material. Use versions from the
repository manifests and lockfiles; Python is pinned in `backend/.python-version`.

The repository originated from the Full-Stack AI Agent Template. Platform and
template-inherited subsystems have different coverage gates, described below.

## Quality and scope

- Type owned boundaries and nontrivial helpers. Do not use `Any` or suppressions to
  hide errors. Necessary suppressions need a specific explanation; route return
  annotations below are an explicit project convention.
- Use domain exceptions with `message` and `details`. Do not swallow exceptions
  or mask bugs with fallbacks.
- Keep changes scoped. Avoid speculative abstractions, unused code and unrelated
  cleanup. Comments explain non-obvious constraints; API contracts belong in
  docstrings. See `.claude/rules/code-style.md`.
- Test new behaviour and add regression tests for bugs. Documentation-only changes
  need documentation checks, not artificial application tests.
- Integrate new user-facing features with the existing product: onboarding for new
  pages and creation flows, and dashboard widgets for activity or state that belongs
  at a glance. Follow `.claude/rules/frontend.md` for registries, permissions,
  translations and anchors.

## Hard boundaries

- Routes call services; services coordinate repositories. Routes must not import or
  call repositories directly.
- Repositories use `db.flush()` and `db.refresh()`, never `db.commit()`. Use
  `DBSession`: its `scope="function"` commits after the route returns and before
  the response is written. A bare `Depends(get_db_session)` has different timing.
  The agent run paths in `AgentRunnerService._run` and `ChatAgentRunner.run`
  explicitly commit before the model call and in terminal cleanup, and
  `SessionService.detect_refresh_reuse` commits the security response its caller
  is about to raise past. See `docs/architecture.md#the-requests-transaction`.
- Dispatch background work needing rows written by the request with
  `spawn_after_commit`, so its own session can see those rows. See
  `docs/architecture.md#dispatching-background-work-from-a-request`.
- Collection routes use `require(...)` gates. Per-resource routes for agents, skills
  and collections delegate to a service using `resolve_access`; a role gate must
  not reject access allowed by a resource grant. Follow `permissions-rbac`.
- Route handlers return `-> Any` and declare `response_model` for serialization.
  Keep service and repository return types precise.
- Store credentials through `app/core/vault.py`. Connector sources reference vault
  secret IDs; do not put credentials in a connector's `CONFIG_MODEL` or introduce
  deployment-wide Fernet keys, `CHANNEL_ENCRYPTION_KEY` or `app/core/crypto.py`.
- Organization authority comes from membership and the permission catalog. Do not
  restore `UserRole`, `User.has_role()`, `RoleChecker`, `CurrentAdmin`,
  `CurrentSuperuser` or the old `users.role` column.
- Use `datetime.now(UTC)` and `secrets.compare_digest()` for API key comparisons.
- Migration numbering restarted after a baseline squash. Verify references against
  `backend/alembic/versions/` and cite full filenames; use git history for removed
  revisions rather than assuming an old number still identifies the same migration.

## Read the matching rule before writing code

Read the relevant files under `.claude/rules/`; their frontmatter defines the file
scope. Load only the rules that apply to the change.

| Editing | Read |
|---|---|
| Any `backend/app/**` Python | `architecture.md` — Routes → Services → Repositories, DI, thin vs. thick domains |
| `schemas/`, `db/models/` | `schemas-models.md` — `*Create`/`*Update`/`*Read`/`*List`, SQLAlchemy |
| `api/` | `api-conventions.md` — REST structure, status codes, pagination, auth aliases |
| `core/`, `services/` | `exceptions-security.md` — domain exceptions, JWT, the permission model |
| Any Python | `code-style.md` — formatting, naming, imports, type hints |
| `tests/` | `testing.md` — the layers, anyio, fixtures, the 100% gate |
| `frontend/` | `frontend.md` — App Router, data layer, stores, i18n, permissions |

## Read the matching skill before starting a task

Skills live in `.claude/skills/<name>/SKILL.md`. Read the skills matching the task;
`.claude/README.md` explains the layout.

| Doing | Skill |
|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vstorm-co/agenticos](https://github.com/vstorm-co/agenticos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
