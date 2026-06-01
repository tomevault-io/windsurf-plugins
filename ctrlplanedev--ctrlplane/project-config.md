---
trigger: always_on
description: This is the canonical agent guidance for this repository. Legacy `CLAUDE.md`
---

# AGENTS.md

This is the canonical agent guidance for this repository. Legacy `CLAUDE.md`
files may still exist, but prefer this file for Codex and other coding agents.

## Product context

Ctrlplane is the orchestration layer between CI/CD pipelines and infrastructure.
It decides when releases are ready, where they should deploy, and which gates
must pass first, including environment promotion, verification, approvals, and
rollbacks.

```text
CI/CD -> Ctrlplane orchestrates -> Infrastructure
```

## Repository overview

- Monorepo managed by Turborepo and pnpm.
- Package names generally use the `@ctrlplane/` scope.
- Primary roots: `apps/`, `packages/`, `integrations/`, `docs/`, `e2e/`,
  `tooling/`.
- `apps/web/`: React 19 and React Router frontend.
- `apps/api/`: TypeScript API, tRPC endpoint, REST/webhook handlers, jsonnet.
- `apps/workspace-engine/`: Go reconciliation engine. See its scoped
  `AGENTS.md` before editing inside that directory.
- `apps/relay/`: Go WebSocket relay for job agent communication.
- `apps/workspace-engine-router/`: Go service for workspace-engine routing.
- `packages/db/`: Drizzle ORM schema, PostgreSQL migrations, database package.
- `packages/trpc/`: tRPC server setup.
- `packages/auth/`: better-auth integration.
- `packages/workspace-engine-sdk/`: TypeScript SDK for external integrations.
- `integrations/`: external service adapters.
- `e2e/`: Playwright end-to-end tests for API and UI flows.
- `tooling/`: shared ESLint, Prettier, TypeScript, OpenAPI, and Tailwind config.

## Setup

- Use `pnpm` for Node/TypeScript work.
- Install dependencies with `pnpm install`.
- Start local services with `docker compose -f docker-compose.dev.yaml up -d`.
- Apply database migrations with `pnpm -F @ctrlplane/db migrate`.
- Start dev servers with `pnpm dev`.
- First-time setup commonly uses:

```bash
docker compose -f docker-compose.dev.yaml up -d
pnpm install
pnpm build
pnpm -F @ctrlplane/db migrate
pnpm dev
```

- To reset local Docker state, wipe volumes first, then remigrate:

```bash
docker compose -f docker-compose.dev.yaml down -v
docker compose -f docker-compose.dev.yaml up -d
pnpm -F @ctrlplane/db migrate
pnpm dev
```

## Common commands

- `pnpm build`: build all packages.
- `pnpm lint`: run ESLint.
- `pnpm lint:fix`: run ESLint with auto-fix.
- `pnpm format`: check formatting.
- `pnpm format:fix`: fix formatting.
- `pnpm typecheck`: type check all packages.
- `pnpm test`: run all tests.
- `pnpm -F <package-name> test`: run tests for a specific package.
- `pnpm -F <package-name> test -- -t "test name"`: run a specific test.

## Database commands

- `pnpm -F @ctrlplane/db migrate`: run migrations.
- `pnpm -F @ctrlplane/db push`: apply schema changes in dev without creating a
  migration file.
- `pnpm -F @ctrlplane/db studio`: open Drizzle Studio.

## E2E tests

- Run all E2E tests from `e2e/` with `pnpm exec playwright test`.
- Run a specific E2E file with
  `pnpm exec playwright test tests/api/resources.spec.ts`.
- Run API E2E tests with `pnpm test:api`.
- Run in debug mode with `pnpm test:debug`.
- E2E tests use YAML fixture files (`.spec.yaml` beside `.spec.ts`) to declare
  entities.
- Use `importEntitiesFromYaml` to load fixtures and `cleanupImportedEntities`
  to tear them down.
- Use `addRandomPrefix: true` when parallel runs could conflict.

## Service architecture

- Web -> API: tRPC over `/api/trpc`.
- API -> workspace-engine: PostgreSQL work queue using `reconcile_work_scope`.
- Relay -> job agents: bidirectional WebSocket streaming.
- External webhooks from GitHub, ArgoCD, Terraform Cloud, and related systems
  enter through `apps/api`.

## Release and deployment flow

```text
CI registers version
-> Release target planning across resources and environments
-> Policy evaluation for approvals, environment ordering, windows, rollout
-> Job dispatch to GitHub Actions, ArgoCD, Terraform Cloud, or custom agents
-> Verification using metrics checks
-> Promotion, retry, or rollback
```

## Reconciliation model

- The workspace-engine uses a PostgreSQL-backed queue.
- Controllers poll `reconcile_work_scope` for leased work items.
- Lease-based locking prevents duplicate processing and allows crashed workers
  to be recovered by another worker after lease expiration.
- Controllers can use `Result.RequeueAfter` for scheduled retries and polling.
- The engine is horizontally scalable.
- The `SERVICES` environment variable can activate a subset of controllers per
  instance.

Core controllers include:

- `deploymentplan`: compute resources that match a deployment selector and fan
  out planning work.
- `deploymentplanresult`: materialize final release rows from plan results.
- `desiredrelease`: determine the target release per resource/environment.
- `policyeval`: evaluate policy rules against release targets.
- `jobdispatch`: route jobs to the correct job agent.
- `jobeligibility`: check whether a job can run.
- `jobverificationmetric`: poll verification metrics.
- `environmentresourceselectoreval`: evaluate environment-level resource
  selectors.
- `deploymentresourceselectoreval`: evaluate deployment-level resource
  selectors.
- `relationshipeval`: evaluate resource relationship rules.
- `forcedeploy`: handle manual force-deploy requests.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ctrlplanedev/ctrlplane](https://github.com/ctrlplanedev/ctrlplane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
