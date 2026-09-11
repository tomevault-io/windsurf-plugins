---
trigger: always_on
description: Full architecture documentation: **[docs/architecture.md](docs/architecture.md)**
---

# Koan Architecture Invariants

Full architecture documentation: **[docs/architecture.md](docs/architecture.md)**

## Frontend Design System (read before any frontend work)

The frontend uses a strict token-driven component system. Visual identity
is user-controlled — agents implement it but do not change it without
approval. Violations compound: a misplaced color becomes a wrong token
becomes an inconsistent component becomes a broken design language.

**When touching any file under `frontend/`**, read
**[frontend/AGENTS.md](frontend/AGENTS.md)** first. It defines protected
files, the component hierarchy (atoms → molecules → organisms), and CSS
conventions.

**When building or modifying a UI component**, also read
**[frontend/src/components/AGENTS.md](frontend/src/components/AGENTS.md)**.
It contains the development rules, the tier decision tree, and the
verification checklist.

---

Spoke documents:

- [docs/agent-protocol.md](docs/agent-protocol.md) -- Agent Protocol, AgentOptions, PydanticAIAgent, steering integration, provider adapter
- [docs/subagents.md](docs/subagents.md) -- spawn lifecycle, task manifest, step-first workflow, permissions
- [docs/initiative.md](docs/initiative.md) -- initiative workflow contract, band hierarchy, gating
- [docs/ipc.md](docs/ipc.md) -- in-process tool calls, blocking interactions, scout spawning, terminal-text hand-back
- [docs/state.md](docs/state.md) -- driver/LLM boundary, run state, orchestrator state
- [docs/intake-loop.md](docs/intake-loop.md) -- two-step intake design, prompt engineering
- [docs/phase-trust.md](docs/phase-trust.md) -- phase trust model, verification boundaries, adversarial review
- [docs/projections.md](docs/projections.md) -- versioned event log, fold function, projection shape, SSE protocol, version-negotiated catch-up
- [docs/token-streaming.md](docs/token-streaming.md) -- in-process StreamEvent delta path, SSE bridge
- [docs/milestones.md](docs/milestones.md) -- milestone soundness criteria, sizing heuristics, grounding requirements
- [docs/workflow-phases.md](docs/workflow-phases.md) -- phase taxonomy across all workflows, producer-validator pairing

**Workflow types:** `plan` (intake -> plan-spec -> plan-review -> execute -> exec-review -> curation) . `milestones` (intake -> milestone-spec -> [milestone-review] -> plan-spec -> [plan-review] -> execute -> exec-review -> milestone-spec loop -> curation) . `initiative` (intake -> core-flows -> tech-plan-spec -> tech-plan-review -> milestone-spec -> [milestone-review] -> plan-spec -> [plan-review] -> execute -> exec-review -> milestone-spec loop -> curation) . `discovery` (frame; single-phase exploration)

---

The six core invariants (see architecture.md for full detail + pitfalls):

**Provider credential model:** Provider availability is `ProviderStatus`
(env-key presence). There is no binary probe. The all-providers model registry
(`ModelRegistryEntry`) is built from the genai-prices bundled snapshot joined
with a koan-owned capability table in `koan/agents/model_catalog.py`. Cost
derivation uses `price_for_usage` against the bundled snapshot only.

## 1. File Boundary

LLMs write **markdown files only**. The driver maintains **JSON state files**
internally -- no LLM ever reads or writes a `.json` file. Tool code bridges
both worlds.

## 2. Step-First Workflow Pattern (critical)

The orchestrator runs as an asyncio task inside the single backend process.
Tools are in-process `FunctionToolset`s composed per (role, phase) via
`compose_toolset` in `koan/tools/tool_policy.py`. There is no boot prompt and
no step-advance tool call.

**Step 1 guidance is injected as the first turn prompt.** The loop
(`run_agent_loop` in `koan/agents/loop.py`) bootstraps by calling
`_step_phase_handshake_core` to obtain step 1 guidance and injects it as
the initial user turn. A **turn-outcome resolver** (`resolve_turn_outcome`)
runs at each end-of-turn (terminal-text turn with no outstanding tool calls):

```
Loop injects step 1 guidance as first turn prompt
     | Agent does work, calls tools as needed
     | Agent ends turn in terminal text (no outstanding tool call)
Resolver fires:
  - completion gate fails  -> re-inject the same step
  - more steps remain      -> inject next step guidance
  - steps exhausted, primary agent -> hand back to user
  - steps exhausted, non-primary   -> terminate
```

At the phase boundary, the primary agent calls `koan_suggest_next` to record
the suggested next steps, then ends its turn in terminal text. The loop surfaces
the text and suggestions and parks awaiting the user. The user's reply resumes
the loop. The agent then calls `koan_set_phase` to commit the transition.
Passing `koan_set_phase("done")` ends the workflow (tombstone).

Phase-specific role context (`SYSTEM_PROMPT`) is prepended to the step 1
guidance at the top of the first turn. Step progression is normally linear
within a phase, but phase modules may override `get_next_step()` to implement
non-linear flows. See [docs/intake-loop.md](docs/intake-loop.md).

Executor subagents are spawned by the orchestrator via `koan_request_executor`.
Scout subagents are spawned via `koan_request_scouts`.

## 3. Driver Determinism (partially relaxed)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [solatis/koan](https://github.com/solatis/koan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
