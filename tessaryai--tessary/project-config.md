---
trigger: always_on
description: An agent-reliability platform: ingest every trace, filter it with cheap classifiers, group the
---

# tessary

An agent-reliability platform: ingest every trace, filter it with cheap classifiers, group the
survivors into cases, and explain a case with an agentic root-cause run grounded in the customer's
own repository. The `.tessary/` pipeline bundle authored by the **evals plugin** (a Claude Code
plugin in a separate repo) is an INPUT — it names call sites, failure modes and intent, and Layer-2
triage rules against it. Nothing in this tree synthesises, runs or scores a grader: this repo has
no graders, datasets, experiments, review queues, or observer. Details:
[`devdocs/reference/architecture.md`](./devdocs/reference/architecture.md).

The product thesis lives in Tessary's internal *Agent Reliability* document,
not in this repo — see *Strategic context* at the bottom for the working summary.
[`devdocs/reference/principles.md`](./devdocs/reference/principles.md) carries the standing
engineering constraints, and [`devdocs/README.md`](./devdocs/README.md) maps the rest of the docs.

## Architecture

```
                     ┌──────────────────────────────┐
                     │ Caddy (:8000) — reverse proxy │
                     └──────┬─────────────────┬─────┘
                  /api/*    │                 │   /
                            ▼                 ▼
              ┌──────────────────┐   ┌──────────────────┐
              │ Spring Boot      │   │ Vite dev (:5173) │
              │ backend (:8080)  │   │ or static build  │
              │ — JVM + Loom     │   │ React + TS       │
              └────┬─────────────┘   └──────────────────┘
                   │ reads / writes            + classify-service (encoder
                   ▼                             /classify heads; separate deploy)
                Postgres (pgvector; per-project pipeline, substrate, findings, cases)
```

| Layer | Tech |
|---|---|
| Reverse proxy | Caddy (`Caddyfile`) — `/api/*`, `/auth/*`, `/mcp` → backend; rest → Vite/static |
| Backend | Spring Boot 4.0.x, Java 25 + Loom virtual threads, an eleven-module Maven reactor (layering in [`devdocs/modules.md`](./devdocs/modules.md)), LangChain4j, Postgres via JdbcClient + Liquibase |
| Frontend | React 19, Vite, TypeScript, TanStack Query, react-router-dom 7, Tailwind v4 (token-driven design system) |
| Auth | WorkOS AuthKit (sealed cookie session); per-project bearer tokens / API keys for MCP + headless ([`devdocs/reference/auth-and-mcp.md`](./devdocs/reference/auth-and-mcp.md)) |

## Repo map

| Path | What it is |
|---|---|
| [`backend/`](./backend/) | The Java backend — conventions in [`backend/AGENTS.md`](./backend/AGENTS.md), inventory in [`devdocs/reference/architecture.md`](./devdocs/reference/architecture.md), module layering in [`devdocs/modules.md`](./devdocs/modules.md) |
| [`frontend/`](./frontend/) | The React app — conventions in [`frontend/AGENTS.md`](./frontend/AGENTS.md) |
| [`classify-service/`](./classify-service/) | Standalone encoder `/classify` service (ECS Fargate) — see its README |
| [`sandbox-runner/`](./sandbox-runner/) | The launcher that runs every agentic lane (RCA, Layer-2 triage) in a fresh E2B microVM — see its README |
| [`classifiers/`](./classifiers/) | The Python classifier tree: the shared eval framework, the `tool_error` and `metric_drift` rigs that check the open Java detectors, and the corpus emitters |
| [`contract/`](./contract/) | Vendored evals-synth output contract (`scripts/sync-evals-contract.sh`). Files are verbatim copies; `contract/tests/` is OURS — the gate for the vendored validator, since the plugin repo is public and runs no CI |
| [`claude-skill/`](./claude-skill/) | Claude Code integration helpers (the MCP skill + prompt-craft reference) |
| [`docs/`](./docs/) | Reference, concepts, guides — start at [`devdocs/README.md`](./devdocs/README.md) |
| `scripts/`, `observability/` | The shared check and deploy scripts; Grafana dashboards |

**Before editing under `backend/` or `frontend/`, read that directory's `AGENTS.md`.** Running
the stack locally: [`devdocs/guides/local-dev.md`](./devdocs/guides/local-dev.md). Tessary's own
hosted deployment is documented privately and is not part of the public export. Cross-stack recipes (schema
change, new classifier): [`devdocs/guides/common-tasks.md`](./devdocs/guides/common-tasks.md).
Config keys: [`devdocs/reference/config-keys.md`](./devdocs/reference/config-keys.md).

## Universal rules

- **No comments in code changes** unless they explain a non-obvious invariant, a workaround
  for a specific bug, or behavior that would surprise a reader. Well-named identifiers are the
  documentation.
- **NEVER keep old implementation as fallback.** Delete dead code.
- **No example files or demo code** unless explicitly requested.
- **User-visible copy follows [`handbook/voice-and-tone.md`](./handbook/voice-and-tone.md).**
  Plain words, active voice, no em-dashes, no hedging. Applies to UI strings, errors, Slack, CLI
  output and docs prose; not to code comments.
- **Do not add or modify tests** unless explicitly requested.
- **The schema is the source of truth.** The evals plugin owns the bundle schema; absorb changes
  in order: `contract/` → backend records → frontend types → views. The plugin still emits grader

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tessaryai/tessary](https://github.com/tessaryai/tessary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
