---
trigger: always_on
description: Agent orchestration loop system built on [pi](https://pi.dev) as the runtime.
---

# Helix

Agent orchestration loop system built on [pi](https://pi.dev) as the runtime.

> **Status:** Experimental. M1 + M2 shipped; Manage + Phase A repo bootstrap landed. **Not for production.**  
> Package: `@eimg/helix` · command: `helix`. See [`docs/plan.md`](./docs/plan.md).

## What Helix is

Helix takes a work item (inline task, local issue tracker webhook, or GitHub issue) and autonomously drives it through user-defined **specialist agents** (planner, dev, verifier, …). A hybrid **orchestrator** coordinates them: reading the issue, deciding which to invoke and in what order, composing each specialist's handoff context, reading results, looping or escalating as needed, and producing a reviewable deliverable (branch/PR when GitHub deliverables are wired).

Helix is **not** an LLM and **not** a coding agent. It is the *system that orchestrates* coding agents. pi is the agent runtime; Helix embeds it in-process via the pi SDK.

**Primary demo path today:** [local-issues](https://github.com/eimg/local-issues) (local tracker → `POST /runs`) + `helix serve`. GitHub/`gh` remains supported but is optional.

## Core loop

```
Trigger (local-issues | inline | GitHub) ─► Orchestrator (hybrid: config workflow + LLM)
                                              │
         ┌────────────────────────────────────┼──────────────────────┐
         ▼                                    ▼                      ▼
   Specialist A (own session, model)   Specialist B (…)        …
         └────────────────── results ─────────┴──────────────────────┘
                              │  reads results, decides: loop / proceed / escalate / done
                              ▼
                   Deliverable (optional PR) + Run state persisted
```

- **Specialists run in parallel** when the orchestrator chooses, each as an **isolated in-process pi session** with its own context window, model, and tools.
- **Specialists never communicate with each other.** The orchestrator is the *only* coordination point.
- **One process.** Parallelism = `Promise.all` over in-process `createAgentSession()` sessions. Subprocess isolation is a future option for untrusted agents, out of scope for now.

## Hybrid orchestrator

The orchestrator is neither a fixed pipeline nor a free-form LLM:

1. **Workflow in config** — declares specialists, a default sequence (e.g. planner → dev → verifier), loop rules (verifier fail → back to dev, max N retries), and merge gate thresholds.
2. **An LLM orchestrator** reasons *within* that scaffold: reads the issue, decides which specialists to actually invoke (may skip, reorder, parallelize), composes handoff prompts, decides loop-vs-escalate.
3. **Deterministic gates** enforce what the LLM must not own: iteration caps, mandatory human approval above merge thresholds, never auto-merge without a passing verifier.

The config gives rails; the LLM adapts to the task; code enforces safety.

## Architecture

| Area | Decision |
|---|---|
| Language | TypeScript (ESM, Node ≥ 20) |
| Package | `@eimg/helix` — npm-installable; binary remains `helix` |
| Agent runtime | pi SDK in-process (`createAgentSession`), not subprocesses |
| Parallel specialists | Isolated sessions; no inter-specialist communication; orchestrator-only coordination |
| Orchestrator | Hybrid: config workflow (rails) + LLM driver (adapts) + deterministic gates (safety) |
| LLM provider | Pluggable; OpenRouter only for v1 |
| Triggers | Inline (CLI/UI); HTTP `POST /runs` (used by [local-issues](https://github.com/eimg/local-issues)); GitHub `gh` + optional poll |
| Config / agents / skills | Repo-local, version-controlled under `.helix/` |
| Skills | Standard pi `SKILL.md`; `.helix/skills/` always loaded into specialist sessions |
| Specialist agents | Markdown + frontmatter (name, description, model, tools, system prompt) |
| Run state | File-based, one JSON per run under `.helix/runs/` |
| Repo context | Phase A: deterministic bootstrap + allowlisted docs injected into first specialist wave |
| Merge gate | Small + verified → auto-merge; big/risky → human approval via API/UI |
| Deliverable | Opt-in GitHub PR via `deliverable.pr` (default off); otherwise no-op after run |
| Web UI | Run console + experimental Manage; consumers of engine event stream + HTTP API |

## Folder layout

```
helix/                          # engine + shipped presets
  AGENTS.md
  docs/plan.md
  src/
    cli.ts                      # `helix` entry (init / run / serve)
    config.ts                   # .helix/config.json loader/validator
    config/paths.ts             # ~/.pi/ resolution for auth/models fallback
    context/                    # Phase A repo bootstrap
    callbacks/                  # issue-tracker completion webhook (POC)
    engine/                     # core loop, event stream, console logger, types
    orchestrator/               # workflow loader + LLM driver + gates + scripted (tests)
    providers/                  # OpenRouter (real) + Fake (tests)
    triggers/                   # GitHub (gh) + poll + inline
    agents/                     # specialist loader, session factory, loader builder, stub (tests)
    deliverable/                # git diff stats, PR create/merge

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eimg/helix](https://github.com/eimg/helix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
