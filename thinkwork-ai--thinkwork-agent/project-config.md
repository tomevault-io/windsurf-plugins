---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Repository at a glance

Thinkwork is an AWS-native agent harness: a TypeScript monorepo plus a Pi AgentCore runtime, deployed by the repo's own CLI via Terraform. There is **no local-only mode** — end-to-end work requires a deployed AWS stack. "Thinkwork supersedes maniflow" — ignore the old `maniflow*` names you may see on stale resources.

- `apps/web` — React 19 + Vite + TanStack Router unified web/operator app (dev port **5174** by default; **5180** is the Cognito callback-friendly worktree port)
- `apps/mobile` — Expo + React Native + NativeWind (iOS via TestFlight)
- `apps/cli` — `thinkwork-cli` (commander.js), published to npm, bundles Terraform modules
- `packages/database-pg` — Drizzle schema + migrations + canonical GraphQL source (`graphql/types/*.graphql`)
- `packages/api` — GraphQL (Yoga) resolvers, Lambda handlers, AppSync subscription bridge
- `packages/lambda` — additional Lambda handlers (job-schedule-manager, job-trigger, agentcore-admin, github-workspace)
- `packages/agentcore-pi` — active AgentCore Pi runtime (Bedrock models, MCP tools, Docker image)
- Tenant S3 skill catalogs — per-tenant folders at `tenants/<tenant-slug>/skill-catalog/<skill-slug>/`; installed skills materialize into workspace `skills/<slug>/` folders
- `packages/system-workspace` — canonical workspace defaults (CAPABILITIES/GUARDRAILS/PLATFORM/MEMORY_GUIDE)
- `terraform/modules/{foundation,data,app,thinkwork}` — three-tier Terraform Registry modules (`thinkwork-ai/thinkwork/aws`)
- `docs/` — planning + institutional knowledge (the docs.thinkwork.ai Starlight site was retired in THINK-702; user-facing docs now live in-app under `apps/web/src/docs`, and the old site content survives unpublished under `docs/reference/`); also holds `plans/`, `brainstorms/`, and `solutions/`; `docs/solutions/` contains documented bugs, patterns, workflow issues, and decisions organized by category with YAML frontmatter (`module`, `problem_type`, `tags`), relevant when implementing or debugging in documented areas
- `CONCEPTS.md` — shared domain vocabulary (entities, named processes, status concepts); relevant when orienting to the codebase or discussing domain terms

## Tooling ground rules

- **pnpm ≥ 9, Node ≥ 20. Never use `npm` inside this workspace** — scripts assume pnpm's workspace protocol. `npx` is fine for one-off CLI tools.
- **Python ≥ 3.11 with `uv`** only for remaining Python helpers/tests. Ruff is the linter (line-length 100, target `py311`). The active AgentCore runtime is TypeScript under `packages/agentcore-pi`.
- **Terraform ≥ 1.5 (or OpenTofu ≥ 1.6)**. Modules are registry-shaped; most real changes happen under `terraform/modules/`.
- Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, `test:`).
- CI runs against Apache-2.0-licensed code; new contributors must sign the CLA on their first PR (CLA Assistant bot).

## Common commands

Run from the repo root unless noted.

### Install / formatting / type / lint (monorepo-wide)

```bash
pnpm install
pnpm -r --if-present build       # or: pnpm build
pnpm -r --if-present typecheck
pnpm -r --if-present lint
pnpm -r --if-present test
pnpm format        # prettier write
pnpm format:check  # prettier check
```

Per-workspace scripts are in each `package.json`. CLI's "lint" is a no-op stub — don't expect ESLint there.

### Running a single test

- **TypeScript (vitest)** — from a package dir: `npx vitest run path/to/file.test.ts` or `npx vitest run -t "test name"`. Suite locations vary: `packages/api` uses `src/**/*.test.ts` **and** `test/integration/**/*.test.ts`; `apps/cli` uses `__tests__/**/*.test.ts`.
- **Python (pytest)** — from repo root: `uv run --with pytest pytest <path>::test_<case>`. `pyproject.toml` limits `testpaths` to `packages/` and the deployment-control-plane module.

### Database / GraphQL schema

Canonical GraphQL lives under `packages/database-pg/graphql/types/*.graphql`. Two schemas are derived from it:

```bash
pnpm schema:build       # regenerates terraform/schema.graphql (AppSync subscription-only schema)
pnpm --filter @thinkwork/database-pg db:generate   # new Drizzle migration from schema changes
pnpm db:push -- --stage dev                         # push Drizzle schema to Aurora (resolves via terraform outputs + Secrets Manager)
pnpm db:migrate-manual                              # drift reporter for hand-rolled .sql files in drizzle/ that aren't in meta/_journal.json
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thinkwork-ai/thinkwork-agent](https://github.com/thinkwork-ai/thinkwork-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
