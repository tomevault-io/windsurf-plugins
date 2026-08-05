---
trigger: always_on
description: > **TEMPLATE — DOMAIN-NEUTRAL.** This repo ships the portable Planner-Worker-Judge workflow with **no** target domain.
---

# AGENTS.md — <PROJECT_NAME>

> **TEMPLATE — DOMAIN-NEUTRAL.** This repo ships the portable Planner-Worker-Judge workflow with **no** target domain.
> Replace every `<PLACEHOLDER>` and every block tagged `> _EXAMPLE_` with your project's real values when you port this setup.
> **Agents:** if a section below is still a `<PLACEHOLDER>`, treat that fact as *unknown* — gather it from the codebase or ask the orchestrator. **Never invent stack, structure, or compliance facts from an example block.**

> **How to use this file:** This is the canonical domain-config hub for the agentic workflow.
> Every agent reads this file at task start. Generic workflow rules, hooks, and skills live in `.cursor/` and do not need editing when you port this setup to a new repo.
> Only this file and the config JSONs below need updating when adding new features:
> - `.cursor/config/protected-paths.json` → `projectProtectedGlobs`
> - `.cursor/config/worker-scopes.json` → `agents` section
> - `.cursor/skills/skills-manifest.json` → add/remove domain skills

---

**IMPORTANT** : Always follow the development rules during the coding phase. See `./docs/development-rules.md`.

## 1. Project Overview

| Field | Value |
|---|---|
| **Project Name** | `<name>` |
| **Domain** | `<domain / industry>` |
| **Description** | `<one-sentence description of what the product does>` |
| **Monorepo** | `<yes (tool: nx / turbo / pnpm-workspaces) | no>` |
| **Multi-Tenancy** | `<none | soft (tenant column) | hard (schema/RLS per tenant)>` |

---

## 2. Tech Stack (Locked — ADR required to change)

> Fill one row per layer your project uses; delete rows that don't apply. State a version only when it is actually pinned. Once filled, changing a locked choice requires an ADR in `docs/adr/`.

| Layer | Technology | Notes |
|---|---|---|
| Language | `<e.g. TypeScript (strict)>` | |
| Backend framework | `<...>` | |
| Frontend framework | `<...>` | |
| Shared contracts | `<schema/validation lib, e.g. Zod>` | source of truth for API types |
| Database | `<...>` | |
| ORM / Migrations | `<...>` | migrations only — no auto-sync |
| Cache / Queue | `<...>` | |
| AI / LLM | `<provider(s) | none>` | |
| Object storage | `<...>` | |
| Auth | `<...>` | |
| Email / Notifications | `<...>` | |
| Payments | `<... | none>` | |
| Infra / Deploy | `<...>` | |
| Testing | `<unit / integration / e2e frameworks>` | |

---

## 3. Repository Structure

> Describe the actual layout of THIS repo. Keep it in sync with `.cursor/config/worker-scopes.json` (agent path scopes must match real folders).

```
<root>/
├── <app-or-package-1>/        # <role>
├── <app-or-package-2>/        # <role>
├── docs/                      # plans, adr, reviews, architecture
└── .cursor/                   # workflow: agents, skills, hooks, config
```

> _EXAMPLE_ (delete when porting): a monorepo might use `apps/api`, `apps/web`, `apps/worker`, `packages/shared-types`. Whatever you choose, mirror it exactly in `worker-scopes.json`.

---

## 4. Multi-Tenancy Rules

> **Applies only if §1 Multi-Tenancy ≠ `none`.** If single-tenant, write "N/A — single-tenant" and skip the guard requirements below.

### Tenant Isolation Pattern

Choose and document your tenant column name (e.g. `tenant_id`, `workspace_id`, `org_id`). Every table that belongs to a tenant **MUST** carry `<TENANT_COL> NOT NULL` with an FK to the tenant table.

- **Tenant-scoped tables:** `<list here>`
- **Global / non-tenant tables:** `<list here>`

### Tenant Guard (mandatory on every tenant-scoped query)

> _EXAMPLE_ pattern — adapt to your framework/ORM:

```typescript
// Every tenant-scoped read MUST filter by the tenant column and select explicit columns:
async findRecords(tenantId: string): Promise<Record[]> {
  return this.repo.find({
    where: { tenantId },            // ALWAYS filter by tenant
    select: ['id', 'name', 'status', 'createdAt'], // NEVER select *
  });
}
```

> ❗ **NEVER write a tenant-scoped query without a tenant filter — no exceptions, not even in admin convenience methods, unless an explicit, logged override flag is used.**

---

## 5. Agent Roster & Scopes

> Roles below are the portable defaults shipped in `.cursor/agents/`. **Path scopes and skills are defined in `.cursor/config/worker-scopes.json` and `.cursor/skills/skills-manifest.json`** — keep those two files as the source of truth and update this table to match. Remove agents you don't use.

| Agent | Role | Scope source | Skills source |
|---|---|---|---|
| `architect-planner` | Plan, ADR, task breakdown, scope definition | `worker-scopes.json` | `skills-manifest.json` |
| `scaffold-agent` | Bootstrap new module/page shells; update §3 paths | `worker-scopes.json` | `skills-manifest.json` |
| `designer-worker` | UI/UX design, component specs, design tokens | `worker-scopes.json` | `skills-manifest.json` |
| `backend-worker` | API features, services, DTOs, guards | `worker-scopes.json` | `skills-manifest.json` |
| `frontend-worker` | Pages, forms, data fetching, client state | `worker-scopes.json` | `skills-manifest.json` |
| `database-worker` | Migrations, entities, query optimization | `worker-scopes.json` | `skills-manifest.json` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phuoctrung-ppt/ai-sdlc-workflow](https://github.com/phuoctrung-ppt/ai-sdlc-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
