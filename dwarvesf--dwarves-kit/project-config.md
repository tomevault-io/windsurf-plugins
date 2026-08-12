---
trigger: always_on
description: > Tool-agnostic front door. Any agent runtime (Claude Code, Codex, Gemini, a
---

# AGENTS.md: the operating layer

> Tool-agnostic front door. Any agent runtime (Claude Code, Codex, Gemini, a
> human) reads this first to learn how work is done in this repo. It carries the
> portable operate-contract: what to read, how to run a unit of work, when work
> is done, and when to stop and ask.

## Enforcement boundary (read this first)

**This file is a contract, not a guardrail.** Enforcement is Claude-Code-only: the
hooks (safety-gate, push-to-main blocker, anti-rationalization Stop hook, the
verification pipeline) are what actually block bad outcomes, and they run only
under Claude Code. Under any other runtime (Codex, Gemini, a bare LLM)
`AGENTS.md` is **advisory only**: it tells an agent what to do, but nothing
enforces it. Do not assume the guardrails are portable. They are not, until the
v3.x multi-runtime agent-hook work lands. See `docs/PHILOSOPHY.md` (honesty rule:
never over-claim portable enforcement) and `CLAUDE.md` for the CC-specific layer
(hooks, slash commands, plugin). The full layering contract, which hook
backstops what and why (hard / advisory / convenience), is
`docs/architecture.md` "Hook fallback layer".

The rest of this file is four portable zones. The goal-crafter
(`commands/assign.md`) projects them into a six-section `/goal` (see "How a goal
is composed" at the end).

---

## 1. Read in this order

Orient before you touch anything. Read top to bottom; stop when you have enough.

1. **AGENTS.md** (this file) - how work is done here; the operate-contract.
2. **CLAUDE.md** - the Claude-Code layer: stack, structure, rules, hooks, commands, plugin.
3. **docs/specs/SPEC-NNN-<slug>.md** - the active spec; the shared contract for the cycle. Read its `## Verification` and `## After state` before implementing.
4. **docs/architecture.md** / **WORKFLOW.md** / **docs/data-flow.md** - reference, not required per task. Read `docs/architecture.md` for how the pieces fit; `docs/data-flow.md` for how DATA moves (the signal path telemetry -> proposal -> board, the ledger write/read paths, the module map); read `WORKFLOW.md` for the lanes and the gate at each phase boundary, and its "Mega-goal delegate execution" section for how a multi-sub-goal mega-goal run dispatches (delegate mode: one fresh headless session per sub-goal; `/goal` stays the official outer loop either way).

## 2. Task loop

How to do one unit of work. The smallest verifiable increment, verified, committed.

0. **Take work.** Handed a task: use it. **Handed a WAVE (2+ items approved in one
   conversation): enqueue every item as a board row FIRST (queued; the in-flight one
   executing), then pull them one at a time.** Work that never touches the board is
   invisible to the board's state machine even when its runs are ledgered, the gap an
   operator caught live on 2026-06-10 (SPEC-064). Not handed one: pull the board's top queued item,
   `bash lib/board/backlog.sh next`, claim it (goal-registry) and flip it to `claimed` (the
   `/kit:assign --next` flow). The BACKLOG is the board; its Status column is the state
   machine (`queued -> claimed -> speccing -> validated -> executing -> shipped`, + parked/
   dropped). Operator-named work is unchanged; pull is an additional trigger, never a daemon.
1. **Classify the type, then size the lane.** `bash lib/classify/task-type-classify.sh classify "<task>"`
   first: `spec-feature` picks a lane below; any other type (incident / reconcile / operate /
   planning / learning / eval / research / review / doc / migration / data-tool) runs its TYPE LOOP per
   `WORKFLOW.md ## Type loops`, with its executor from the registry's `agent` column. The lane
   is STILL sized for every type (it is the evidence contract ship-gate enforces via the spec's
   `Lane:` header; the type is the content contract , `WORKFLOW.md ### Lane x type composition`). For code:
   pick `tiny` / `normal` / `full` / `bug` / `backfill` per `WORKFLOW.md`; when in doubt between
   two lanes, take the heavier one. **Between classification and done comes the grill** (`/kit:grill`, or its
   one-question-at-a-time discipline driven inline): interview until the task is actually
   understood, type-shaped questions, recommended answers, contradictions checked against the
   repo, answers WRITTEN as they resolve (glossary / sparse ADR / the goal draft's Context).
   Tiny lane exempt. **Record the grill's disposition either way** (SPEC-063):
   `bash lib/gate/gate-ledger.sh record <rid> grill ran "<N> branches resolved"`, or, when the
   conversation already resolved the banks, `... record <rid> grill skipped "<why>"`; a
   skip without a reason is invisible to telemetry, which defeats the point.
   **Then phase 0: define the done scenario**
   (`bash lib/gate/proof-gate.sh contract "<task>"` + the type's test-design dialect,
   test-design-standard §5b) BEFORE any work runs; the grill's answers are the done's raw
   material, and the goal draft carries the `Done =` line.
2. **Read the spec and its acceptance criteria.** For a spec-driven task: the active spec's task row, its AC, its `## Verification`, and its `## After state`. No spec (tiny lane): the one obvious edit.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dwarvesf/dwarves-kit](https://github.com/dwarvesf/dwarves-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
