---
trigger: always_on
description: This repository's long-term project memory lives in `.project-memory/`. Memory uses an "index + chunked body" layout: each topic's `MEMORY.md` only routes, and fact bodies are read on demand.
---

# Project Memory and Workflow Constraints

This repository's long-term project memory lives in `.project-memory/`. Memory uses an "index + chunked body" layout: each topic's `MEMORY.md` only routes, and fact bodies are read on demand.

## Task Routing

- Every new task first uses `read-index-memory` to read all topic indexes, then reads only the bodies relevant to the task; a task that touches an area designed but not built yet also reads that `Plan/` body.
- Pure consultation, code review, or external research does not enter the modification loop; the memory-reading, documentation-lookup and output constraints still apply.
- Adding, fixing, refactoring, configuration, tests, docs or prompt syncs are "modification tasks" and follow the loop below.

## Staged Skill Loading

- Do not load every skill mentioned in this file at task start. Determine the next concrete action and load only the skill needed for that action; do not preload skills for later workflow stages.
- `read-index-memory` is used at task start under Task Routing. Load `post-verify` only after modification work is complete. Load `update-memory` when an eligible durable fact or preference needs recording, and after successful `post-verify` for the required final memory review; do not preload it at task start unless that capture is the next action.
- Load `design-alignment` only when a new design needs user agreement, or a design conflict must be resolved, and the next action is to settle that decision. Do not preload it for ordinary implementation of an already-settled design.
- Load `docs-research`, `collect-update-memory`, and other skills only when their trigger condition is met and the next action requires them. A skill being named in this file is not itself a trigger to load it.

## Memory Capture

- Across all tasks, trigger `update-memory` when a durable project goal, scope, preference or boundary is explicitly stated; do not wait for task completion or technical verification.
- Stable implicit project preferences or boundaries may also trigger capture as soon as the signal recurs across separate task contexts without a contrary correction; do not wait for task completion or technical verification. `update-memory` handles their evidence and inference label.
- Immediately after `post-verify` passes, perform one final memory review before delivery: inspect the completed work for implementation-derived facts, conflicts with existing memory, and execution lessons with confirmed recurrence risk and reusable conditions; record eligible findings with `update-memory`.
- This final review does not delay eligible explicit or inferred preferences and boundaries; record them when they become clear. Do not record temporary progress, unresolved proposals or unsupported technical claims.

## Modification Task Loop (MUST)

### 1. Before modifying

- Read the project memory bodies relevant to the current task and confirm the project boundary, target, design and effective constraints.
- Identify the modification entry point, callers/callees, the affected data or state flow, and the available verification methods.

### 2. Execution

- Use `update_plan` only when the task really has more than three interdependent steps, multiple independent decisions, or otherwise complex decomposition; update it with actual progress and mark completed steps immediately.
- For external documentation and version-sensitive changes, follow the Documentation Lookup Boundary below.
- Reuse existing implementations and conventions; make the smallest change that works; prefer the standard library or existing dependencies; do not refactor unrelated code, drop requirements, or change behavior that was not asked for.
- Keep going on complex tasks until "implemented, results checked, found problems fixed, verification complete" are all done; do not stop early just because the first version is implemented, unless a safety boundary genuinely needs a user decision.

### 3. Verification

- Every completed modification task (including configuration, templates, docs and prompts) must run the `post-verify` skill.

### 4. Memory consolidation

- After verification passes, complete the final memory review described in `Memory Capture` before delivery.

## Plan and Completion Boundaries

- `update_plan` only records the decomposition and status of the current task; it does not write project memory or TODO, nor replace them. The project's design plan is `.project-memory/Plan/` — always write that path in full, never a bare `Plan`.
- The completion standard is decided by the user's request; the default delivery is "result implemented, critical path checked, verification evidence recorded, remaining risks stated".

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [czh020110/Morrowmark](https://github.com/czh020110/Morrowmark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
