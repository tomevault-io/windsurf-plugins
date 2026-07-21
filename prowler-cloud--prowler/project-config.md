---
trigger: always_on
description: - Start here for cross-project norms. Prowler is a monorepo with several components.
---

# Repository Guidelines

## How to Use This Guide

- Start here for cross-project norms. Prowler is a monorepo with several components.
- Each component has an `AGENTS.md` file with specific guidelines (e.g., `api/AGENTS.md`, `ui/AGENTS.md`).
- Component docs override this file when guidance conflicts.

## Available Skills

Use these skills for detailed patterns on-demand:

### Generic Skills (Any Project)

| Skill | Description | URL |
|-------|-------------|-----|
| `typescript` | Const types, flat interfaces, utility types | [SKILL.md](skills/typescript/SKILL.md) |
| `react-19` | No useMemo/useCallback, React Compiler | [SKILL.md](skills/react-19/SKILL.md) |
| `nextjs-16` | App Router, Server Actions, proxy.ts, streaming | [SKILL.md](skills/nextjs-16/SKILL.md) |
| `tailwind-4` | cn() utility, no var() in className | [SKILL.md](skills/tailwind-4/SKILL.md) |
| `playwright` | Page Object Model, MCP workflow, selectors | [SKILL.md](skills/playwright/SKILL.md) |
| `pytest` | Fixtures, mocking, markers, parametrize | [SKILL.md](skills/pytest/SKILL.md) |
| `django-drf` | ViewSets, Serializers, Filters | [SKILL.md](skills/django-drf/SKILL.md) |
| `jsonapi` | Strict JSON:API v1.1 spec compliance | [SKILL.md](skills/jsonapi/SKILL.md) |
| `zod-4` | New API (z.email(), z.uuid()) | [SKILL.md](skills/zod-4/SKILL.md) |
| `zustand-5` | Persist, selectors, slices | [SKILL.md](skills/zustand-5/SKILL.md) |
| `ai-sdk-5` | UIMessage, streaming, LangChain | [SKILL.md](skills/ai-sdk-5/SKILL.md) |
| `vitest` | Unit testing, React Testing Library | [SKILL.md](skills/vitest/SKILL.md) |
| `tdd` | Test-Driven Development workflow | [SKILL.md](skills/tdd/SKILL.md) |

### Prowler-Specific Skills

| Skill | Description | URL |
|-------|-------------|-----|
| `prowler` | Project overview, component navigation | [SKILL.md](skills/prowler/SKILL.md) |
| `prowler-api` | Django + RLS + JSON:API patterns | [SKILL.md](skills/prowler-api/SKILL.md) |
| `prowler-ui` | Next.js + shadcn conventions | [SKILL.md](skills/prowler-ui/SKILL.md) |
| `prowler-sdk-check` | Create new security checks | [SKILL.md](skills/prowler-sdk-check/SKILL.md) |
| `prowler-mcp` | MCP server tools and models | [SKILL.md](skills/prowler-mcp/SKILL.md) |
| `prowler-test-sdk` | SDK testing (pytest + moto) | [SKILL.md](skills/prowler-test-sdk/SKILL.md) |
| `prowler-test-api` | API testing (pytest-django + RLS) | [SKILL.md](skills/prowler-test-api/SKILL.md) |
| `prowler-test-ui` | E2E testing (Playwright) | [SKILL.md](skills/prowler-test-ui/SKILL.md) |
| `prowler-compliance` | Compliance framework structure | [SKILL.md](skills/prowler-compliance/SKILL.md) |
| `prowler-compliance-review` | Review compliance framework PRs | [SKILL.md](skills/prowler-compliance-review/SKILL.md) |
| `prowler-provider` | Add new cloud providers | [SKILL.md](skills/prowler-provider/SKILL.md) |
| `prowler-changelog` | Changelog entries (keepachangelog.com) | [SKILL.md](skills/prowler-changelog/SKILL.md) |
| `prowler-ci` | CI checks and PR gates (GitHub Actions) | [SKILL.md](skills/prowler-ci/SKILL.md) |
| `prowler-commit` | Professional commits (conventional-commits) | [SKILL.md](skills/prowler-commit/SKILL.md) |
| `prowler-pr` | Pull request conventions | [SKILL.md](skills/prowler-pr/SKILL.md) |
| `prowler-docs` | Documentation style guide | [SKILL.md](skills/prowler-docs/SKILL.md) |
| `django-migration-psql` | Django migration best practices for PostgreSQL | [SKILL.md](skills/django-migration-psql/SKILL.md) |
| `postgresql-indexing` | PostgreSQL indexing, EXPLAIN, monitoring, maintenance | [SKILL.md](skills/postgresql-indexing/SKILL.md) |
| `prowler-attack-paths-query` | Create Attack Paths openCypher queries | [SKILL.md](skills/prowler-attack-paths-query/SKILL.md) |
| `prowler-tour` | Keep product-tour definitions aligned with the UI | [SKILL.md](skills/prowler-tour/SKILL.md) |
| `gh-aw` | GitHub Agentic Workflows (gh-aw) | [SKILL.md](skills/gh-aw/SKILL.md) |
| `skill-creator` | Create new AI agent skills | [SKILL.md](skills/skill-creator/SKILL.md) |

### Auto-invoke Skills

When performing these actions, ALWAYS invoke the corresponding skill FIRST:

| Action | Skill |
|--------|-------|
| Add changelog entry for a PR or feature | `prowler-changelog` |
| Adding DRF pagination or permissions | `django-drf` |
| Adding a compliance output formatter (per-provider class + table dispatcher) | `prowler-compliance` |
| Adding indexes or constraints to database tables | `django-migration-psql` |
| Adding new providers | `prowler-provider` |
| Adding privilege escalation detection queries | `prowler-attack-paths-query` |
| Adding services to existing providers | `prowler-provider` |
| Adding, updating, or removing a tour definition (*.tour.ts) | `prowler-tour` |
| After creating/modifying a skill | `skill-sync` |
| App Router / Server Actions | `nextjs-16` |
| Auditing check-to-requirement mappings as a cloud auditor | `prowler-compliance` |
| Building AI chat features | `ai-sdk-5` |
| Changing button labels or section headings on a tour-covered page | `prowler-tour` |
| Committing changes | `prowler-commit` |
| Configuring MCP servers in agentic workflows | `gh-aw` |
| Create PR that requires changelog entry | `prowler-changelog` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prowler-cloud/prowler](https://github.com/prowler-cloud/prowler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
