---
trigger: always_on
description: Use when the user wants a long-running implementation run with phase gates, durable progress tracking, subagent execution, review checkpoints, or explicit wording like "$10ximplement", "10x implement", "run this plan", "phase-gate this build", "orchestrate builders", "keep implementation notes", "track decisions", or "continue this implementation". Combines a goal ledger, PM phase gates, subagent-driven plan execution, and executing-plans fallback into one workflow.
---


# 10ximplement

Use this skill to run implementation like an orchestrated build: keep durable state, choose narrow phases, execute through builders or direct plan steps, review before each gate, and leave the next agent a clear resume point.

## Core Contract

- Keep `.agent/runs/<goal-id>/implementation-notes.html` as the canonical current truth for the run.
- Keep `.agent/runs/<goal-id>/GOAL.md` as the contract: objective, finishing criteria, parent goal, native goal coupling line if relevant, and escape hatch.
- Keep `.agent/GOALS.md` as the project-level run index.
- Use `assets/implementation-notes.template.html` and `scripts/init_10x_ledger.py` when starting a new run.
- Update the HTML after phase gates, validation, blockers, accepted defers, handoffs, meaningful implementation checkpoints, and final completion.
- Ask at phase boundaries when the user must choose PM or review mode. Once a plan execution mode is chosen, execute continuously until blocked, all tasks complete, or the gate requires a decision.

## Start Or Resume

1. Inspect current context: user request, repo docs/specs, latest handoff or existing `.agent/` run, and `git status --short`.
2. If no ledger exists for this run, create one:

```bash
python3 <skill-dir>/scripts/init_10x_ledger.py \
  --root . \
  --goal-id "<stable-goal-id>" \
  --title "<short title>" \
  --objective "<one sentence objective>" \
  --mode full
```

3. If a ledger already exists, read `GOAL.md` and the top `Resume Here` section before touching code.
4. Only create or update a native runtime goal when the user explicitly asks for goal mode, `$goal`, `/goal mode`, "start a goal", or when a native runtime goal already exists. Put the ledger path in that runtime goal objective.
5. Define or confirm finishing criteria before implementation.

## Phase Gate Loop

Repeat this loop until the goal is complete:

1. **Load context:** read the canonical product/spec entrypoint, latest memory/handoff, current build spine, only relevant domain docs, and changed files.
2. **Review previous phase:** verify what was supposed to be true, what exists, what checks pass, and which issues are blockers versus defers.
3. **Choose next slice:** pick one narrow vertical phase with product value and stable architecture. Name non-goals. Avoid mixing unrelated infrastructure, billing, provider, messaging, and UI polish unless the phase explicitly requires it.
4. **Select PM mode if not already chosen for this phase:**
   - Traditional PM Mode: write a compact builder prompt or phase brief for external implementation. Do not edit code directly.
   - Inline PM Mode: run the phase in this chat. Make only small low-risk edits yourself; use builder subagents for larger, risky, or multi-file work.
5. **Prepare the phase brief:** include working directory, files/docs to read, build scope, non-goals, required checks from existing project scripts, stop condition, and expected summary.
6. **Execute or hand off:** use the execution mode below.
7. **Review gate:** run the selected review mode, verify findings, run checks, decide accept checkpoint, request fixes, shrink scope, or redirect.
8. **Ledger update:** record status, decisions, logic, validation, blockers, accepted defers, protected paths, evidence links, and next exact action in `implementation-notes.html`.

## Execution Modes

Use the mode that fits the phase:

- **Subagent-driven execution:** Use when a written plan exists, tasks are mostly independent, and subagents are available in the current session. Dispatch a fresh implementer per task, then spec compliance review, then code quality review. Do not move to the next task while either review has open issues.
- **Executing-plans fallback:** Use when a written plan exists but subagents are unavailable, tasks are tightly coupled, or execution is happening as a direct sequential session. Read the plan, review it critically, follow every task exactly, run stated verification, and stop on blockers or repeated verification failures.
- **PM handoff:** Use in Traditional PM Mode. Write the builder prompt/phase brief and wait for returned builder output before reviewing.
- **Small direct edit:** Use in Inline PM Mode only for narrow, low-risk changes. Still review and update the ledger.

Before executing implementation plans, respect branch safety: do not start implementation on `main` or `master` without explicit user consent. Prefer an existing isolated worktree; do not fight a platform-managed workspace.

## Subagent Rules

When using subagents:

- Provide full task text and curated context. Do not make subagents read the whole plan to discover their task.
- Use one implementer per task unless a PM phase has clean, non-overlapping builder ownership lanes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cobibean/10ximplement](https://github.com/cobibean/10ximplement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
