---
trigger: always_on
description: This repository is a multi-tenant enterprise AI platform. Optimize for correct, minimal, verifiable, production-safe changes. The default bar is enterprise-grade, not demo-grade.
---

# AgenticOrg Claude Guide

This repository is a multi-tenant enterprise AI platform. Optimize for correct, minimal, verifiable, production-safe changes. The default bar is enterprise-grade, not demo-grade.

## MANDATORY for bug fixes and reopen analysis

Every bug fix, reopen triage, QA-driven PR, or bug-list engagement on this
repo MUST use the repo-authored skill
[`/.claude/skills/agenticorg-bug-fix-fail-closed/SKILL.md`](.claude/skills/agenticorg-bug-fix-fail-closed/SKILL.md)
and follow the canonical checklist in
[`docs/bug_triage_skill.md`](docs/bug_triage_skill.md). The skill is the
entrypoint; `docs/bug_triage_skill.md` is the source of truth for the
fail-closed verdict matrix, symptom-grep rule, sibling-path sweep,
test-replays-tester's-steps rule, merged-vs-deployed honesty, release
sign-off discipline, and Alembic safety gates. Producing a bug-fix summary,
reopen verdict, or release sign-off without walking that checklist is an
incorrect work product.

Forbidden verdicts: "should be fixed", "the code looks correct", "probably a
cache issue", "fixed in main" (without deploy confirmation).

## MANDATORY for enterprise hardening, architecture review, and release sign-off

Any whole-codebase audit, enterprise hardening review, production-readiness
assessment, architecture-safety review, or release sign-off in this repo MUST
use the repo-authored skill
[`/.claude/skills/agenticorg-enterprise/SKILL.md`](.claude/skills/agenticorg-enterprise/SKILL.md).
This includes reviews of workflow durability, auth/session hardening, billing
runtime behavior, connector secret handling, async safety, startup DDL,
multi-tenant isolation, and deployment readiness. Producing an enterprise
verdict or release sign-off without walking that skill is an incorrect work
product.

If the task is both bug-related and enterprise-hardening-related, use both
repo-authored skills together: `agenticorg-bug-fix-fail-closed` and
`agenticorg-enterprise`.

## Mission

- Deliver the simplest correct change that satisfies the request.
- Never weaken security, tenancy isolation, secrets handling, or operability for convenience.
- Prefer clarity over cleverness, explicitness over magic, and small diffs over broad rewrites.

## Repo Context

- Backend: FastAPI, async SQLAlchemy, Alembic, Redis, Celery, LangGraph.
- Frontend: React 19, TypeScript, Vite, Vitest, Playwright.
- Infra: Docker, Docker Compose, Helm, GitHub Actions.
- Core directories:
  - `api/` request handlers and FastAPI app wiring
  - `auth/` authn/authz and middleware
  - `core/` business logic, models, tool gateway, billing, tasks
  - `migrations/` schema migrations
  - `ui/` frontend app
  - `helm/`, `docker-compose.yml`, `.github/workflows/` deployment and delivery

## Operating Principles

### 1. Think Before Coding

- Do not silently choose an interpretation when the request is ambiguous.
- State assumptions explicitly before making consequential changes.
- If there is a simpler or safer approach than the requested one, say so.
- If a behavior, schema, contract, or security boundary is unclear, inspect first and ask only if needed.

### 2. Simplicity First

- Build the minimum code that solves the actual problem.
- Do not add speculative abstractions, options, flags, or indirection.
- Do not introduce a framework pattern for a one-off use case.
- If a solution feels "future-proof" but the future is hypothetical, cut it.

### 3. Surgical Diffs

- Touch only files and lines that are necessary.
- Match the existing style and local patterns unless the task is explicitly a refactor.
- Do not opportunistically rewrite adjacent code, comments, or formatting.
- Only remove dead code if your change made it dead or the user asked for cleanup.

### 4. Goal-Driven Execution

- Translate requests into explicit success criteria.
- Prefer verifiable outcomes over vague implementation work.
- For non-trivial work, form a short plan with checks.
- Keep going until the change is implemented and verified, unless blocked by missing information or permissions.

## Non-Negotiable Enterprise Rules

### Authz and Tenancy

- Never trust `tenant_id`, role, domain, or privilege claims from the client request body or query string when an authenticated server-side context exists.
- Bind tenant-scoped operations to authenticated tenant context on the server.
- Any tenant-wide admin or control-plane action must enforce server-side authorization, not only frontend role gating.
- UI route guards are convenience only. Backend authorization is the real control boundary.
- Fail closed on missing user, missing tenant, incomplete session hydration, or ambiguous privilege state.
- For multi-tenant code, actively test the "tenant A trying to affect tenant B" case in your head before shipping.

### Secrets and Sensitive Data

- Do not store secrets in plaintext config, plaintext database fields, logs, metrics, or exceptions.
- If a secret must be persisted, prefer encrypted storage or a real secret manager path.
- PII masking is for logs, traces, and audit artifacts, not for live execution payloads.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mishrasanjeev/agentic-org](https://github.com/mishrasanjeev/agentic-org) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
