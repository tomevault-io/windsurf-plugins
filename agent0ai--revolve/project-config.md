---
trigger: always_on
description: Revolve is an instruction-only protocol for reproducible self-improvement.
---

 # Revolve Protocol

Revolve is an instruction-only protocol for reproducible self-improvement.

Use for improve/optimize/evolve/research/benchmark/tune/self-improve requests. For one-offs, answer normally. For iterative work, create or resume `revolve/`.

## Source Principles

This protocol is self-contained. Preserve reproducible research principles over runtime history.

## Research Progress

- Reliability gates are necessary, not sufficient. Passing reliability gates makes a candidate eligible; Do not stop at a merely passing candidate unless the objective, user, or budget says so.
- Research Momentum: a green run is evidence, not a stop condition. Continue from the strongest unresolved opportunity until a meaningful result exists: validated improvement, documented dead end, revised objective/evaluation proposal, or explicit stop; avoid cosmetic churn.
- Iteration must follow evidence: each continuation names an observed failure, opportunity, tradeoff, or uncertainty; the next checkpoint hypothesis; and measurable expected effect.
- Quality Search Batch: for quality objectives, compare an organization branch, navigation branch, and research-momentum branch; choose the next branch from evidence.
- Research Scope: tune a local research copy under `revolve/`; external promotion is optional, not the default goal, and live codebase changes require user intent.
- Internal Incumbent Discipline: track the current best checkpoint, promote internally when evidence passes, base new work on the best comparable checkpoint, do not reset to the original baseline after promotion, and keep external promotion separate.
- External Promotion Boundary: internal promotion is the research default; external promotion is a separate live-file decision; live artifacts remain unchanged unless explicitly promoted.
- Branch Portfolio: keep a leaderboard and Promising Branch Queue; rank by evidence, record selection reason, revisit older promising branches when newer branches stagnate, revisit when active line stalls, and do not flatten research into one round.
- Checkpoint Depth Loop: create the next checkpoint on the same branch, fork from an earlier checkpoint, or repeat or refine before accepting a shallow pass.
- Navigation Quality: parent indexes expose the resume-critical decision, what to open next, why branches stopped, and enough detail to avoid rereading raw history.

## Core Rules

- Evaluation first: define or connect the evaluation environment before changing the subject.
- Checkpoint first: preserve the incumbent subject before candidate work.
- Local research first: tune checkpointed subject copies under `revolve/`; live artifacts stay read-only unless external promotion is explicit.
- Same revision, same comparison: directly compare scores only inside one revision.
- New evaluation, new revision: create/select a new revision when cases, scoring, harness, evaluator rubric, subject definition, objective interpretation, or acceptance gates change.
- Parent docs route; child docs explain.
- Parent indexes expose resume-critical state: active child, current best, tried branch statuses, blocker, and next action; child docs keep detail.
- Documentation gates are blocking: do not advance until required local and parent `AGENTS.md` updates for the current state are complete.
- Do not keep one giant research diary. Store durable state in local `AGENTS.md` files.
- Do not silently mutate an evaluated checkpoint. Any meaningful subject change creates a new checkpoint.
- Do not leave evaluated checkpoints or branches marked `pending` or `active` unless awaiting a run or documented next action.
- Do not overfit. The main agent must not use benchmark memorization, evaluation-set leakage, exact-answer lookup, or test-case-specific hacks to raise a score.
- Promote only with acceptance evidence, or explicit user choice with documented tradeoffs and rollback.
- Treat subject failures, harness failures, infrastructure failures, and dataset gaps as different things.
- If sub-agents are available, the main agent owns synchronization, parent docs, revisions, incumbent changes, promotion, and live-file edits.
- Sub-agents document inside assigned output folders and do not mutate shared parent state.
- Ask blocking questions only; proceed with documented assumptions.

## Terms

- Main agent: user-facing owner of intent, projects, revisions, integration, parent docs, promotion, live changes.
- Sub-agent: bounded worker; explores, evaluates, analyzes, or proposes, then reports without shared-state changes.
- Subject: improved prompt, code, config, workflow, policy, dataset, model, evaluator, or artifact.
- Evaluation environment: harness, cases, fixtures, scoring, acceptance gates, result schema, and run procedure.
- Harness: subject evaluator: code, commands, tests, benchmark, LLM review, human review, or hybrid.
- Case: reproducible scenario with input, expected behavior or metric, and pass/fail criteria.
- Checkpoint: recoverable state with parent, rationale, results, status, and restore method.
- Branch: line of checkpoints pursuing one search hypothesis.
- Revision: versioned evaluation context; scores are comparable only within the same revision.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agent0ai/revolve](https://github.com/agent0ai/revolve) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
