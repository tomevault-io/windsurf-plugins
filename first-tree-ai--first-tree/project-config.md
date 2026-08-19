---
trigger: always_on
description: **first-tree** — the unified CLI and infrastructure for agent teams. It is a pnpm monorepo for the CLI, server, client runtime, web app, docs, and agent-team tooling.
---

# AGENTS.md

**first-tree** — the unified CLI and infrastructure for agent teams. It is a pnpm monorepo for the CLI, server, client runtime, web app, docs, and agent-team tooling.

What first-tree is NOT:

- not an LLM agent itself (agent logic lives elsewhere)
- not an orchestration framework

## Tech Stack

- **Server:** Fastify / Drizzle ORM / PostgreSQL / Zod
- **Client:** fetch + ws (Cloud SDK/observability + generic Runtime + built-in Providers)
- **Command:** Commander.js / @inquirer/prompts (unified CLI)
- **Shared:** Zod schemas + TypeScript types + config system
- **Web:** React 19 / Vite
- **Tooling:** pnpm / Turborepo / Biome / Vitest / tsdown
- **Node.js:** minimum 22.13, recommended 24

## Common Commands

```bash
pnpm install
docker compose up -d

pnpm --filter @first-tree/server dev
pnpm --filter @first-tree/web dev

pnpm check && pnpm typecheck
pnpm test

pnpm --filter @first-tree/server db:generate
pnpm --filter @first-tree/server db:migrate
```

Full CLI commands and env vars live in [docs/cli-reference.md](docs/cli-reference.md). Per-package scripts live in each package's `package.json`.

## Required Reading By Topic

- **HTTP routes, JWT auth, scope helpers, or multi-org behavior:** read [docs/development/http-path-conventions.md](docs/development/http-path-conventions.md) before editing. It is the single source of truth.
- **Onboarding kickoff/chat-start behavior:** read [docs/development/onboarding-kickoff-contract.md](docs/development/onboarding-kickoff-contract.md) before editing `/me/onboarding/kickoff`, `/orgs/:orgId/context-tree/setup-chat`, or the client inbound prompt formatting.
- **Running an in-tree CLI next to prod/staging:** use `scripts/dev-install.sh` and read [docs/development/local-dev-isolation.md](docs/development/local-dev-isolation.md).

## Repo-Local Skills

- `skills/first-tree-write/SKILL.md` — source-driven Context Tree authorship
- `skills/first-tree-read/SKILL.md` — task-scoped Context Tree reading
- `skills/first-tree-seed/SKILL.md` — one-time bootstrap for an empty tree
- `skills/first-tree-qa/SKILL.md` — risk-tiered professional QA workflow

Operator-only flows such as `login`, `daemon install`, and `agent create` belong in [docs/cli-reference.md](docs/cli-reference.md) and [docs/onboarding-guide.md](docs/onboarding-guide.md), not in skills.

## Monorepo Structure

- `apps/cli/` — unified CLI source; CI publishes channel-specific packages
- `apps/doc-website/` — documentation website
- `packages/shared/` — `@first-tree/shared` schemas, types, and config
- `packages/server/` — `@first-tree/server` Fastify API server
- `packages/client/` — `@first-tree/client` Cloud SDK/observability, generic Runtime, and built-in Providers
- `packages/web/` — `@first-tree/web` React workspace
- `packages/skill-evals/` — eval tooling for repo-local skills
- `packages/qa/` — internal QA workflow assets for agent-run validation
- `docs/` — user, operator, development, migration, and troubleshooting docs
- `skills/` — repo-local skill payloads

## Architecture Rules

- **Package boundaries:** Server, Client, Command, and Web are independently packaged/deployed and share code through `@first-tree/shared`. The CLI is the user-facing command surface and depends only on Client + Shared. Server ships separately as the SaaS Docker image.
- **Server state:** Server is stateless. PostgreSQL is the only persistence/queue/notification backend; do not add Redis or MQ.
- **Unified user-JWT auth:** A single user JWT authorizes Web/Admin API calls and every agent the user manages on the client WebSocket. Route classification, JWT shape, and scope helpers live in [docs/development/http-path-conventions.md](docs/development/http-path-conventions.md). Channel homes live in [docs/development/local-dev-isolation.md](docs/development/local-dev-isolation.md). Agents bind via `agents.client_id` + `agent:pinned`; R-RUN is re-evaluated at every `agent:bind`. Switching users goes through `first-tree login <code>` and the local-client switch path; `logout --purge` retires the current server client and cuts its runtime routes before destructive local cleanup, after which cleared agents can be moved to a new connected runtime from Web.
- **Inbox boundary:** Server writes to Inbox; Client pulls / receives WebSocket notifications. Delivery is at-least-once; Client deduplicates.
- **Agent identity:** Agents are managed by the server Admin API. Agent profile markdown lives in `agents.profile`. Context Tree integration is optional and injected by Client at workspace startup.
- **Credentials:** Sensitive credentials are AES-256-GCM encrypted at the application layer via `services/crypto.ts`.
- **Messages:** Message IDs are UUID v7 and messages are immutable after creation.

## Coding Conventions

- Use `unknown` + type narrowing instead of `any`.
- Avoid `as` assertions; when unavoidable for third-party libraries, explain why nearby.
- Do not use `enum`; use `as const` objects and Zod-compatible literals.
- Use `import type`, prefer `type` over `interface` unless extension/implementation requires an interface, and give public APIs explicit return types.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [first-tree-ai/first-tree](https://github.com/first-tree-ai/first-tree) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
