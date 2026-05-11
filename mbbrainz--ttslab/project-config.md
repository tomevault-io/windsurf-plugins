---
trigger: always_on
description: > **MANDATORY: This project has Lattice initialized (`.lattice/` exists). You MUST use Lattice to track all work. Creating tasks, updating statuses, and following the workflow below is not optional — it is a hard requirement. Failure to track work in Lattice is a coordination failure: other agents and humans cannot see, build on, or trust untracked work. If you are about to write code and no Lattice task exists for it, stop and create one first.**
---

# ttslab

## Lattice

> **MANDATORY: This project has Lattice initialized (`.lattice/` exists). You MUST use Lattice to track all work. Creating tasks, updating statuses, and following the workflow below is not optional — it is a hard requirement. Failure to track work in Lattice is a coordination failure: other agents and humans cannot see, build on, or trust untracked work. If you are about to write code and no Lattice task exists for it, stop and create one first.**

Lattice is file-based, event-sourced task tracking built for minds that think in tokens and act in tool calls. The `.lattice/` directory is the coordination state — it lives alongside the code, not behind an API.

### Creating Tasks (Non-Negotiable)

Before you plan, implement, or touch a single file — the task must exist in Lattice. This is the first thing you do when work arrives.

```
lattice create "<title>" --actor agent:<your-id>
```

**Create a task for:** Any work that will produce commits — features, bugs, refactors, cleanup, pivots.

**Skip task creation only when:** The work is a sub-step of a task you're already tracking (lint fixes within your feature, test adjustments from your change), pure research with no deliverable, or work explicitly scoped under an existing task.

When in doubt, create the task. A small task costs nothing. Lost visibility costs everything.

**Recurring observations become tasks.** If you observe the same issue in 2+ consecutive sessions or advances (e.g., a failing test, a lint warning, a flaky behavior), create a task for it. Agents are disciplined about tracking assigned work but not discovered work — this convention closes that gap. Create discovered issues at `needs_human` if they need scoping, or `backlog` if they're well-understood.

### Descriptions Carry Context

Descriptions tell *what* and *why*. Plan files tell *how*.

- **Fully specified** (bug located, fix named, files identified): still go through `in_planning`, but the plan can be a single line (e.g., "Fix the typo on line 77"). Mark `complexity: low`.
- **Clear goal, open implementation**: go through `in_planning`. The agent figures out the approach and writes a substantive plan.
- **Decision context from conversations**: bake decisions and rationale into the description — without it, the next agent re-derives what was already decided.

### Status Transitions

Every transition is an immutable, attributed event. **The cardinal rule: update status BEFORE you start the work, not after.** If the board says `backlog` but you're actively working, the board is lying and every mind reading it makes decisions on false information.

```
lattice status <task> <status> --actor agent:<your-id>
```

```
backlog → in_planning → planned → in_progress → review → done
                                       ↕            ↕
                                    blocked      needs_human
```

**Transition discipline:**
- `in_planning` — before you open the first file to read. Then write the plan.
- `planned` — only after the plan file has real content.
- `in_progress` — before you write the first line of code.
- `review` — when implementation is complete, before review starts. Then actually review.
- `done` — only after a review has been performed and recorded.
- Spawning a sub-agent? Update status in the parent context first.

### Sub-Agent Execution Model

Each lifecycle stage gets its own sub-agent with fresh context. This is the default execution pattern — not a suggestion, not complexity-gated. Every task, every time.

**Why this matters:** When a planning agent writes a plan and a separate implementation agent reads it, the plan *must* be clear and complete — there's no shared context to fall back on. This forces better plans. When a review agent reads the diff cold, it catches things the implementer's context-polluted mind would miss. The plan file and git diff are the handoff artifacts.

**The three sub-agents:**

| Stage | Sub-agent does | Reads | Produces |
|-------|---------------|-------|----------|
| **Plan** | Explore codebase, write plan, move to `planned` | Task description | Plan file |
| **Implement** | Read plan, build it, test, commit, move to `review` | Plan file | Committed code |
| **Review** | Read diff cold, review against acceptance criteria, record findings | Git diff + plan | Review comment (`--role review`), move to `done` |

**The parent orchestrator** (the main agent session) manages the lifecycle:
1. Move the task to `in_planning` before spawning the planning sub-agent.
2. After the planner finishes, move to `in_progress` and spawn the implementation sub-agent.
3. After the implementer finishes, the review sub-agent runs independently.

Each sub-agent should use a distinct actor ID (e.g., `agent:claude-opus-4-planner`, `agent:claude-opus-4-impl`, `agent:claude-opus-4-reviewer`) so the event log shows who did what.

### The Planning Gate


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MbBrainz/ttslab](https://github.com/MbBrainz/ttslab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
