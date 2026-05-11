---
trigger: always_on
description: This document defines how this project is planned, designed, implemented, and validated. It provides precise conventions the team and agents must follow so that work stays traceable end‑to‑end and easy to navigate.
---

# Agents Guide: Delivery, Design, Tasks, Tests, and Linking

This document defines how this project is planned, designed, implemented, and validated. It provides precise conventions the team and agents must follow so that work stays traceable end‑to‑end and easy to navigate.

Scope
- Top‑down delivery flow from backlog to code and tests
- Folder and file naming conventions
- Tech design expectations and diagramming
- MECE task breakdowns with exhaustive testing
- WAV output generation for test cross‑checks
- Cross‑linking rules for all docs

## Time & Complexity Assumptions
- Agents operate without time limits or compute ceilings; always pursue the correct solution rather than deferring work due to perceived constraints.
- Prefer deeper debugging and full validation over partial fixes or relaxed tests. Any mention of time pressure should be corrected to reflect these assumptions.
- Do not cite time or complexity limits as blockers; when a problem is large, plan and execute incrementally while maintaining the expectation of full completion.


Related entry points
- Backlog: `docs/delivery/backlog.md`
- Master technical design: `docs/tech-design.md`
- UAT spec: `docs/user-acceptance-testing.md`


## Backlog Management

Authoritative backlog lives in `docs/delivery/backlog.md`. It is ordered top‑to‑bottom by priority. The numeric ID is unique and stable but is not the priority.

Rules
- Add new items to the bottom of the table by default.
- Assign a new unique integer ID. Do not reuse or renumber.
- New items default to no Epic (Epic may be assigned later).
- Write backlog story titles in clear plain-English so musicians and integrators understand them at a glance while keeping the behavior precise.
- Reprioritization is done by reordering rows only; never by changing IDs.
- Each backlog ID must have a corresponding folder: `docs/delivery/<id>/`.
- Status tracking is required in the backlog: Maintain a visible status per item (Proposed | In Progress | Done). Determine status by inspecting the item’s tech tasks index (`docs/delivery/<id>/tech-tasks.md`):
  - Proposed — no `tech-tasks.md` or the index exists but contains no tasks
  - In Progress — `tech-tasks.md` exists and at least one task is not Done
  - Done — `tech-tasks.md` exists and all tasks are Done
  Maintain a single backlog table with a Status column. Do not add separate summary tables; the main table is the source of truth.
- Whenever a story’s status changes (e.g., after updating tasks), update the `Status` column in `docs/delivery/backlog.md` and every occurrence of that story in `docs/user-acceptance-testing.md`, including scenario tables and the “Overall UAT status” cells.

When adding a new backlog item
1) Append a new row to `docs/delivery/backlog.md` with ID, concise user story, optional Epic, and Delivery links:
   - `./<id>/tech-design.md`
   - `./<id>/tech-tasks.md`
2) Create folder `docs/delivery/<id>/` and the two files above.
3) Start with status index in `tech-tasks.md` (see “Tech Tasks Index”).


## Backlog Item Folder Layout

For a backlog item with ID `<id>`:
- `docs/delivery/<id>/tech-design.md` — exhaustive design for this item, aligned to the master design.
- `docs/delivery/<id>/tech-tasks.md` — index of the implementation tasks with status.
- `docs/delivery/<id>/<id>-<task>.md` — one file per task, MECE and test‑complete.

Naming
- `<id>` is the backlog ID (e.g., `31`).
- `<task>` is a unique task index for the item (e.g., `1`, `2`, `3`).
- Examples in this repo: `1-1.md`, `33-4.md`, `49-2.md`, etc.


## Tech Design Expectations

Each item’s `tech-design.md` describes the delta from the master architecture in `docs/tech-design.md` in enough detail that implementation is mechanical. It must:
- Start with a short plain-English introduction (2-3 sentences) that explains what changes and why it matters for users.
- The introduction must explicitly call out the user-facing problem and the proposed solution so reviewers can verify scope at a glance.
- At the end of implementing a story, double-check the story’s `tech-design.md` introduction still does this plain-English problem/solution summary before marking the work complete.
- Link back to the backlog row and forward to the tasks index.
- Enumerate all affected components, modules, and public APIs.
- Include mermaid diagrams where relevant:
  - Component/struct/module relationships
  - Sequence diagrams for the golden path, alternate flows, and error flows
 - Specify invariants, pre/post‑conditions, failure modes, and error codes.
 - Define config/parameters and their valid domains, ranges, and units.
 - Call out performance budgets and real‑time constraints. Any change that can touch the audio render loop or other hot-path code must include a performance impact assessment (benchmarks or profiling) before it ships.
- Quantify expected inner-loop latency/throughput impact, document acceptable deltas vs baseline, and reference the benchmarks/tests that will enforce them.

Starter header for `docs/delivery/<id>/tech-design.md`:

```markdown
# <Title>: Tech Design for <id>


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [boxabirds/maw](https://github.com/boxabirds/maw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
