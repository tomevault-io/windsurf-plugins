---
trigger: always_on
description: High-level guide for coding agents working in this repository.
---

# AGENTS.md

High-level guide for coding agents working in this repository.

## Product scope

Multi-tenant LLM observability platform. The repo is a **pnpm** workspace orchestrated with **Turbo**.

At a glance: **`apps/*`** own HTTP boundaries (validation, authz, routing to use-cases); **`packages/domain/*`** own business rules and ports; **`packages/platform/*`** implement infrastructure adapters; **`@repo/utils`** holds cross-cutting pure helpers. Telemetry and control data flow through **Postgres**, **ClickHouse**, **Redis**, and object storage, with **organization-scoped** access everywhere at the boundary.

## Repo-wide conventions

- Organization-scoped Redis or cache keys must start with the organization prefix: `org:${organizationId}:...`. Put the org id first so tenancy is obvious and keyspaces stay consistently partitioned.
- Never invoke `tsc` directly. Typechecking goes through `tsgo` via the package `typecheck` script — use `pnpm --filter <pkg> typecheck` for one package or `pnpm typecheck` for the whole workspace. `tsc` would diverge from CI.
- ClickHouse migrations must be created with `pnpm --filter @platform/db-clickhouse ch:create <migration_name>`. Do not create ClickHouse migration files manually.
- **PR base branch follows the branch's origin.** Branches forked from `development` (v2 work, default) PR into `development`; branches forked from `latitude-v1` (v1 maintenance) PR into `latitude-v1`. Never start a branch from `main` without confirmation — if the working branch is based on `main`, or the user asks to branch from `main`, confirm with the user first. Detect an existing branch's base by testing ancestry in order: if `git merge-base --is-ancestor origin/latitude-v1 HEAD` is true → `latitude-v1`; else if `git merge-base --is-ancestor origin/development HEAD` is true → `development`; else the branch is likely based on `main` (or something unusual) — stop and confirm with the user before opening a PR. When the user asks to start a new branch, take the base from their wording (mentions of v1 → `latitude-v1`; otherwise default to `development`).
- Production deploys use the single-branch release-tag flow: `development` is trunk and deploys to staging by default; production is triggered only by pushing a `vX.Y.Z` tag that points at the latest `origin/development` commit. Before tagging a production release, update `CHANGELOG.md` with a human-readable diff of the code being pushed to production since the previous production deploy, focusing on the major aspects rather than every commit. Use `scripts/release.sh [version]` to fetch and tag the latest `origin/development` commit; without a version it bumps the latest `vX.Y.Z` tag to the next patch version, with `--minor` / `--major` available for larger bumps. Do not promote by merging `development` into `main`.

## How to use this guide

1. Skim the **skill glossary** below and open the skill that matches your task.
2. Read that skill's `SKILL.md` in full before editing code in that area.

Detailed policies, command examples, and code samples live under **`.agents/skills/<skill-name>/SKILL.md`**. Load narrow skills instead of memorizing the entire monorepo at once.

**Index coverage:** The glossary lists **every** skill in `.agents/skills/` (one row per `*/SKILL.md`, **24** total), ordered **alphabetically by folder name**. When you add or remove a skill folder, update this table in the same change.

## Skill glossary

| Skill | Path | Use when |
| --- | --- | --- |
| **Agentation watch mode** | [.agents/skills/agentation-watch-mode/SKILL.md](.agents/skills/agentation-watch-mode/SKILL.md) | Agentation annotation watch loops, continuous feedback handling, or when the user says **`watch mode`** and wants annotations acknowledged, fixed, and resolved as they arrive |
| **Analyze problem** | [.agents/skills/analyze-problem/SKILL.md](.agents/skills/analyze-problem/SKILL.md) | Investigating a bug, task, or reported issue to explain behavior, root cause, proposed fix, and verification steps before implementation |
| **API endpoints (HTTP, MCP, SDK)** | [.agents/skills/api-endpoints/SKILL.md](.agents/skills/api-endpoints/SKILL.md) | Adding or changing routes in **`apps/api`**, **`defineApiEndpoint`**, **`openapi.json` / `mcp.json` regen**, **`createXxxRoutes`** factories, writing **field descriptions** that propagate to both the **TS SDK** and **MCP tool** consumers |
| **Architecture and boundaries** | [.agents/skills/architecture-boundaries/SKILL.md](.agents/skills/architecture-boundaries/SKILL.md) | Layering, web vs public API, **app layout** (clients, routes, logging), ports/adapters, **web-standard APIs in domain/shared/utils**, multi-tenancy, DDD layout, anti-patterns, **machine-facing MCP/API product surfaces** |
| **Background jobs and events** | [.agents/skills/async-jobs-and-events/SKILL.md](.agents/skills/async-jobs-and-events/SKILL.md) | **Queues/workers**, **domain events**, side effects **outside** HTTP handlers, task payload design, debounce/dedupe, delayed job semantics, **domain event naming**, **publisher–consumer decoupling** |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [latitude-dev/latitude-llm](https://github.com/latitude-dev/latitude-llm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
