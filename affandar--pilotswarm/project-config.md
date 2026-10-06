---
trigger: always_on
description: The published `agent-manager` package and agent display as **Agent Smith**.
---

# Copilot Instructions for PilotSwarm

## Agent Smith naming

The published `agent-manager` package and agent display as **Agent Smith**.
Keep the stable `agent-manager` IDs, FQNs, file paths, capability gates, and
provider/session configuration unchanged when updating the display name.
Existing system sessions named Agent Manager use the same display alias.

## Chat call previews

Portal tool and cross-agent calls use compact, collapsible first-line previews.
Correlate execution and external-tool events by their call/request IDs within
the durable session; completion updates the existing row. Keep bulk history,
live append, and backward paging consistent, and preserve disclosure identity.
Render raw arguments/results as text. Empty-response diagnostics remain only
in Activity; do not turn them back into chat warnings. The TUI keeps calls in
its Activity view.

## **⚠️ NEVER commit, push, or deploy without explicit user permission. ALWAYS ask first.**

> **MANDATORY:** Do NOT run `git commit`, `git push`, `git tag`, deploy scripts, or any
> operation that modifies the repository history or remote state without the user
> explicitly saying "commit", "push", or "deploy". Stage files and describe what
> you'd commit, then ask.

## Repo Scope Boundary

When the user asks for changes, releases, or deploys in this repository, operate on `pilotswarm` only unless they explicitly ask to update a downstream consumer as well.

- Do **not** propagate PilotSwarm changes into downstream app repos or vendored SDK mirrors by default.
- Do **not** deploy downstream projects as part of a PilotSwarm deploy unless the user explicitly asks for that separate rollout.
- If a downstream project is out of sync, call it out clearly instead of silently patching it.

## Bug Report Anonymization

When writing or updating files under `docs/bugreports/`, keep downstream deployments anonymous by default.

- Do **not** include downstream app names, customer/project names, proprietary agent names, cluster/context names, namespaces, hostnames, URLs, endpoints, email addresses, or other deployment-specific identifiers unless the user explicitly asks to preserve them.
- Replace live identifiers with clear placeholders such as `<downstream-app>`, `<cluster-context>`, `<namespace>`, `<session-id>`, or generic labels like "downstream app" / "named-agent session".
- Avoid pasting raw production session IDs, pod names, or copied prompt content that would reveal a downstream app's identity when a summarized or redacted example is sufficient.
- Before committing bug-report docs, scan them for identity leaks and redact anything that is not required to explain the bug.

## Sensitive Local Files

**Do NOT modify `.model_providers.json`, `.env`, or `.env.remote` without the user explicitly asking.** The real `.model_providers.json` is a local gitignored config file that may contain personal service URLs, while `.env` / `.env.remote` hold credentials. Never read env files to extract secrets, never commit the real `.model_providers.json`, and never overwrite local config with placeholder content unless the user asked for that exact reset.

When env or model-catalog shape changes, keep `.env.example` and
`.model_providers.example.json` in sync with placeholder values. The catalog is
provider-type/model metadata only. Named shared/personal providers,
credentials, defaults, limits, allowances, and holds are runtime CMS state;
do not add new live credentials or `defaultModel` routing to checked-in
catalogs. Env-backed catalog credentials remain bootstrap compatibility only.

AKS Dockerfiles copy the explicit checked-in deploy catalog, never the private
gitignored `.model_providers.json`. Verify the type catalog inside running
worker/portal images plus the viewer-scoped runtime `listModels()` surface.
Provider/budget schema changes follow the normal stored-procedure migration
rules and must keep management client, Web API, MCP, TUI/portal, docs, and tests
in parity.

`.env` and `.env.horizondb` are two **standalone** local configs, never layered. `.env` is stock-PostgreSQL only: it backs the default `./scripts/run-tests.sh` run (all suites with the PG fact store) and must leave the HorizonDB enhanced-facts/graph settings empty. `.env.horizondb` is a complete standalone config (stock PG runtime storage + HorizonDB enhanced facts/graph) used by `./scripts/run-tests.sh --with-horizondb` and the Horizon Harvester scripts. Those paths load **only** `.env.horizondb` — there is **no fallback to `.env`, ever** — so every value the run needs (`DATABASE_URL`, `GITHUB_TOKEN`, model keys, `HORIZON_*`) must be duplicated in it. Keep `.env.horizondb.example` in sync with its shape, like `.env.example`.

## Project Overview

pilotswarm is a durable execution runtime for [GitHub Copilot SDK](https://github.com/github/copilot-sdk) agents, powered by [duroxide](https://github.com/microsoft/duroxide) (a Rust-based durable orchestration engine). It provides **crash recovery, durable timers, session dehydration, and multi-node scaling**.

## Architecture

The runtime separates into two runtime components:

- **`PilotSwarmClient`** — manages sessions, sends prompts, subscribes to events. Lightweight, no GitHub token needed. Only handles serializable data.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [affandar/PilotSwarm](https://github.com/affandar/PilotSwarm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
