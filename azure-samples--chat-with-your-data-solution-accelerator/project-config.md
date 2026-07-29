---
trigger: always_on
description: These instructions are **always loaded**. Read them before doing anything else.
---

# Chat With Your Data (CWYD) — Repository Instructions

These instructions are **always loaded**. Read them before doing anything else.

## Repository layout (truth)

- `cwyd-v1/` — **v1 (legacy, frozen)**. Flask + Streamlit + Semantic Kernel + Prompt Flow + direct Azure OpenAI SDK. **Do not extend.** Only touch v1 to perform documented removals listed in [v2/docs/development_plan.md](../v2/docs/development_plan.md) §2.1. All v1 code, infra, docker, scripts, and tests live under `cwyd-v1/`.
- Repo root (`src/`, `infra/`, `docker/`, `scripts/`, `tests/`, `azure.yaml`, `pyproject.toml`, `package.json`) — **v2 (active development)**. FastAPI + LangGraph + Agent Framework + Foundry IQ, promoted from the old `v2/` subtree to the repo root. All new work goes here.
- `infra/` (root) — v2 Bicep. The v1 Bicep lives under `cwyd-v1/infra/`.
- `tests/` (root) — v2 tests (shared gates, backend, frontend, functions, infra, integration, smoke). The v1 e2e/integration tests live under `cwyd-v1/tests/`.
- `v2/docs/` — the only tree still under `v2/`: v2 development plan, pillars, ADRs, bugs registry, and worklog.

## Mandatory references — consult before editing

1. [v2/docs/development_plan.md](../v2/docs/development_plan.md) — phase ordering, file paths, scope, removals, additions. Source of truth for **what** to build and **when**. **Always read §0 Status snapshot + the head of §0.1 debt queue before proposing a task.** As of 2026-05-11 the **active phase is Phase 6 — Functions blueprints / modular RAG indexing pipeline** (`batch_start`, `batch_push`, `add_url`, `search_skill` under `src/functions/`; ingestion-only extensions under `src/functions/core/`). Phases 1–5 + 5.5 are closed; #35d (FE admin merge) and #24 partial (FE SSE polish) are open and explicitly **non-blocking** for Phase 6. #35g (per-tenant overrides) is **withdrawn as out of scope** — the deployment is single-tenant, so per-tenant `RuntimeConfig` keying is a no-op; see [ADR 0024](../v2/docs/adr/0024-withdraw-per-tenant-runtime-config-single-tenant.md).
2. [v2/docs/pillars_of_development.md](../v2/docs/pillars_of_development.md) — every new core element must declare its pillar (Stable Core, Scenario Pack, Configuration Layer, or Customization Layer) in the file/class docstring. **Read-only product policy** — never edited by agents.
3. Repo memory `cwyd-tech-stack.md` — current stack truth (versions, services, package managers, test counts).

## External pattern sources — read-only

You may **fetch and read** these for architectural patterns. **Never copy code wholesale** — adapt with attribution in a code comment when the pattern is non-trivial.

- Content Generation Solution Accelerator: <https://github.com/microsoft/content-generation-solution-accelerator>
- Multi-Agent Custom Automation Engine (MACAE): <https://github.com/microsoft/Multi-Agent-Custom-Automation-Engine-Solution-Accelerator>

Specifically:

- MACAE → managed-identity + RBAC + no-Key-Vault env-var pattern; agent-to-agent message bus; SSE streaming pattern.
- CGSA → React/Vite + FastAPI plug-and-play surface; admin merged into frontend; reasoning visualization patterns.

## Hard rules

0. **Step 0 — sync agent guidance before any change.** Before any reorganization, refactor, or error fix, first read the agent instructions and prompt files that scope the change (this file + the relevant `.github/instructions/v2-*.instructions.md`). If the guidance is stale, contradicts the requested change, or is silent on a decision the change implies, **propose an instruction update first and wait for user approval** before touching code or other docs. Out-of-date guidance is the root cause of re-work, throwing away decisions, and re-introducing removed concepts.
1. **One unit per turn.** Implement exactly **one class** OR **one method** per implementation turn. No "and while I'm here…" edits. Do not create multiple files of production code in a single turn unless they are a class + its test stub.
2. **Test-first contract.** Every new method/class lands with at least a `pytest` (Python) or `vitest`/`jest` (TS) test file in the same turn. Tests must execute (pass or fail with a clear assertion), not just exist.
3. **Pillar awareness (no docstring header).** Agents still classify every new module/class by its pillar (Stable Core, Scenario Pack, Configuration Layer, or Customization Layer) when reasoning about where a change belongs — but the classification is **no longer emitted as a `Pillar:` / `Phase:` docstring header**; that header requirement is retired. The pillars file (`v2/docs/pillars_of_development.md`) is **read-only product policy** — agents reference it, never edit it. Any proposed change to pillars must be raised with the user as a separate request.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure-Samples/chat-with-your-data-solution-accelerator](https://github.com/Azure-Samples/chat-with-your-data-solution-accelerator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
