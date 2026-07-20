---
trigger: always_on
description: > **For AI models** (Claude Code, Cursor, future agents) joining this codebase. Read this first to onboard fast. Cross-references are markdown links so they render both on GitHub and in Obsidian.
---

# AGENTS.md — Luper deep reference

> **For AI models** (Claude Code, Cursor, future agents) joining this codebase. Read this first to onboard fast. Cross-references are markdown links so they render both on GitHub and in Obsidian.
>
> Human entry point: [README.md](README.md). Binding design spec: [docs/design_spec.md](docs/design_spec.md). Permanent rationale behind the design: [docs/design_notes.md](docs/design_notes.md).

---

## 0. TL;DR

Luper = a thin Python harness driving the `claude` / `codex` CLIs through a fixed workflow:

```
brief → contract → ✋ approval gate → plan(+critic) → execute_loop(retry/replan/research) → done|partial|error
```

- **No API keys** for the executor agents — uses the local CLIs under the user's subscription.
- **OpenAI API** is used only for Deep Research (`o3-deep-research`, opt-in).
- **Single user, single task at a time, single process.** No multi-tenancy, no DB.
- **State on disk:** `tasks/<id>/state.json` + `events.jsonl` + `llm_calls.jsonl` + artifacts. No SQLite, no LangGraph.
- **Cockpit** = FastAPI + HTMX, a read + control window over `tasks/`.

Current state: **Post-MVP, Cesta A (incremental polishing path), public-release prep underway.** A0–A5 sprints + the post-test-7 batch (P0.1 frontmatter-safe regex, P0.2 monitor de-spam recipe, P1.1 critic caveats, P1.4 artifacts-only finalize) are done. The 2026-05 dual-model audit ([`docs/audit_2026-05_synthesis.md`](docs/audit_2026-05_synthesis.md)) is the basis for the post-audit cleanup PRs sequenced in the internal implementation plan (private archive). The supervisor agent (brief-launch skill) is now the default operating model (see [docs/design_notes.md "Why supervisor agent is now default"](docs/design_notes.md#why-supervisor-agent-is-now-default-post-audit-2026-05-transition)).

---

## 1. Mental model

### 1.1 What this is NOT

- Not an AI app for end users. There is no chat interface, no agent dialogue.
- Not a multi-step agent like AutoGPT — the workflow is **fixed**, not LLM-decided.
- Not a state-machine engine — it's a `while` loop with explicit phases.
- Not a paid-by-call service — uses subscription CLIs and never reports cost.

### 1.2 What it IS

A **harness** (text-defined) that orchestrates a fixed sequence:

1. **Brief** — user writes a markdown task description.
2. **Contract** — `claude` agent reads the brief and produces a structured contract (goal, deliverables, acceptance_criteria).
3. **Approval gate** — runner pauses, user reviews the contract in the cockpit, edits the markdown if needed, clicks Approve.
4. **Plan** — `claude` agent splits the contract into 1–20 steps (each step has its own acceptance_criteria). `codex` agent acts as the critic; the planner finalises.
5. **Execute loop** — for each step:
   - Executor (`claude` or `codex` per step) produces artifact(s).
   - Runner runs **deterministic checks** (word_count, regex_present, min_bytes, citation_count, python_ast_parse).
   - Verifier (`claude` agent) decides pass / fail / inconclusive based on artifacts + deterministic results.
   - On pass: critic loop (codex) reviews the verdict, verifier finalises.
   - On fail / inconclusive: retry up to `max_retry_per_step`. Then replan up to `max_replan_per_task`. Then partial.
6. **Done | Partial | Error | Stopped | Discarded** — terminal state.

The runner is **deterministic** in routing — no LLM decides "what next". Only the agents inside each phase use LLMs.

### 1.3 Workflow as text

The workflow is defined in `workflow/`:

- [workflow/workflow.md](workflow/workflow.md) — human prose description of the phase sequence.
- [workflow/prompts/](workflow/prompts/) — agent system prompts (`contract.md`, `planner.md`, `executor.md`, `verifier.md`, `critic.md`, plus shared `_system_overview.md`).
- [workflow/schemas/](workflow/schemas/) — JSON Schema for `contract.json`, `plan.json`, `verdict.json`.

Each task **snapshots** the entire `workflow/` directory into `tasks/<id>/workflow_snapshot/` at creation time. The runner reads the snapshot, not the live `workflow/`, so a task remains reproducible even if you edit prompts mid-way.

---

## 2. Repo map

```
runner/             async Python orchestrator
cockpit/             FastAPI + HTMX read+control UI
workflow/                  text + JSON: workflow.md, prompts/, schemas/
tasks/                     gitignored runtime state
tests/                     pytest, includes integration suites
docs/                      spec + findings + design notes
```

Key files (with absolute role):

| File | Role | Critical? |
|---|---|---|
| [runner/orchestrator.py](runner/orchestrator.py) | Main `while` loop. Drives the task through phases. Owns state.json. | ⭐⭐⭐ |
| [runner/phases.py](runner/phases.py) | One function per phase: `run_contract`, `run_plan`, `run_executor`, `run_verifier`, `run_research_step`. | ⭐⭐⭐ |
| [runner/cli.py](runner/cli.py) | Single point where LLM subprocess calls happen. Logs everything to `llm_calls.jsonl`. | ⭐⭐⭐ |
| [runner/sessions.py](runner/sessions.py) | Persistent claude session metadata (Sprint 6). Deterministic UUIDv5. | ⭐⭐ |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zisky-ai/Luper](https://github.com/Zisky-ai/Luper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
