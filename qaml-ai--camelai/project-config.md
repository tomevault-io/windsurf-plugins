---
trigger: always_on
description: Keep this file concise and durable. Add details here only when they help future agents navigate the repo or avoid common mistakes. Feature-specific behavior should usually live near the code or in tests.
---

# camelAI Agent Guide

Keep this file concise and durable. Add details here only when they help future agents navigate the repo or avoid common mistakes. Feature-specific behavior should usually live near the code or in tests.

## What This Is

camelAI is an AI coding assistant platform on Cloudflare Workers + Durable Objects with Cloudflare sandbox containers for builds/analysis. Users chat with persistent coding workspaces, run either Claude SDK or Codex app-server backed threads, and publish generated apps to `*.camelai.app` / environment-specific app hosts.

## High-Level Architecture

```text
React Router SSR + browser WS
        |
        v
Cloudflare main Worker + Durable Objects
        |
        v
Project files in WorkspaceFilesystemDO + R2 (do-r2 backend)
Builds/deploys + analysis in Cloudflare sandbox containers

Dispatcher Worker routes published user apps.
R2 stores uploads/assets/previews.
Cloudflare AI Gateway and BYOK credentials back model access.
```

Agent turns run in `ChatThreadDO` (Pi coding agent). Project source files live
in `WorkspaceFilesystemDO` + R2 (every project is `backend: "do-r2"`); builds,
deploys, and analysis run in Cloudflare sandbox containers (`ProjectBuildSandbox`,
`AnalysisSandbox`, `DbQuerySandbox`). The legacy Azure project-runtime-service VM
and its `PROJECT_RUNTIME_HOST` bridge are gone; the only remaining VM is the
static-IP database egress relay (`infra/db-egress-relay/`, see `docs/db-egress-relay.md`).
SQL queries/exports run in `DbQuerySandbox` (the `DATA_PROXY` binding is served
worker-side). There is no in-repo Go sandbox-host or data-proxy tree.

## Repository Map

- `src/` - React Router 7 app, routes, loaders/actions, UI components, shared server/client libraries.
- `src/routes.ts` - Imperative React Router route config. Add page/API routes here.
- `src/routes/api/` - React Router API routes for most user-facing REST (billing checkout, workspaces, chat groups, etc.).
- `src/components/ui/` - shadcn/ui components.
- `workers/main/` - Main Cloudflare Worker, Durable Objects, WebSocket routing, MCP, admin APIs, proxies, container image Dockerfiles.
- `workers/main/src/identity/` - `UserDO` / `OrgDO` and related identity helpers (`auth.ts` is a compatibility barrel).
- `workers/main/src/routes/` - Worker-native HTTP (WebSockets, Stripe webhook, data-proxy, MCP, most `/api/admin/*` on Hono). Prefer documenting new paths here vs `src/routes/api/` — see **API routing** below.
- `workers/dispatcher/` - Workers for Platforms dispatcher for deployed user apps.
- `workers/app-usage-guard/` - Account-wide Durable Object SQLite usage monitor and reversible app quarantine Worker; see `docs/deployed-app-usage-guard-design.md`.
- `workers/bedrock-provider/` - AI Gateway custom provider translating Anthropic-style requests to Bedrock.
- `workers/user-logs-tail/` - Tail worker for deployed app logs.
- `workers/e2e-reports/` - Public viewer at `e2e-reports.camelai.dev` serving Playwright E2E reports from R2 (uploaded by the E2E workflow); deploy with `bun run deploy:e2e-reports`.
- `workers/eval-reports/` - Read-only results store + viewer for agent evals at `evals.camelai.dev` (evals run locally; `EVAL_REPORT=1` publishes them); deploy with `bun run deploy:eval-reports`.
- The Go data-proxy (external `qaml-ai/project-runtime-service` `cmd/data-proxy`) is **retired**: SQL queries and warehouse exports now run in the `DbQuerySandbox` Cloudflare container (`workers/main/src/db-query-service.ts` + `data-proxy.ts` compat surface), and the `SANDBOX_HOST` VPC binding is gone. Do not reintroduce either. Decommission checklist: `docs/db-egress-relay.md`.
- `sandbox/` - Agent skills, project scaffold templates (`create-worker/`), and the canonical `validate-notebook.py` (byte-copied into `workers/main/analysis-sandbox-assets/` for the analysis image build context). Not the agent control plane or harness — those live in `workers/main` (`chat-thread-do.ts`, Pi tools, Dockerfiles).
- `scripts/` - Deploy, eval, self-host, and maintenance scripts. One-off migrations (`migrate-to-workspaces.ts`, `import-legacy-emails.ts`) are break-glass only.
- `docs/` - Supporting documentation; see `docs/README.md` for the canonical index (many `*-plan.md` / feedback files are historical).
- `plans/` - Active cross-cutting architecture plans (e.g. OrgDO split, no-VM build/deploy).
- `infra/` - Terraform for the static-IP database egress relay VM (`infra/db-egress-relay/`); `infra/selfhost/` for self-host cloud templates. See `infra/README.md`.
- `tests/` - Vitest UI / `src/lib` unit tests (`vitest.config.ts`).
- `workers/main/tests/` - Worker / Durable Object / Miniflare tests + `evals/` (`vitest.workers.config.ts`).
- `e2e/` - Playwright end-to-end specs.
- `.claude/skills/` - Agent skills for this repo (evals, shadcn, writing skills).

### API routing

Two HTTP surfaces share the main worker:

| Surface | Location | Typical contents |
| --- | --- | --- |
| React Router | `src/routes/api/` | Session-cookie user REST (workspaces, billing checkout, uploads, chat groups) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qaml-ai/camelAI](https://github.com/qaml-ai/camelAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
