---
trigger: always_on
description: <!-- mine-managed-agents -->
---

<!-- mine-managed-agents -->
# Agent Working Agreement

MINE Is Not Everyone's. These are durable, repository-wide agreements. Package-specific schemas, contracts, implementation decisions, and plan details belong in the design knowledge base or the relevant plan, not here.

## Source of truth

- Design knowledge base root: `docs/design/index.md` (progressive disclosure; indexes orient, leaves specify).
- Design ownership marker: `docs/design/.mine-design.toml`.
- Execution plans: `docs/plan/`.
- Execution graph machine source: `docs/plan/execution-graph.toml`.
- Execution graph generated view: `docs/plan/execution-graph.md`.
- Implementation and review reports: `docs/plan/reports/`.
- Project configuration: `.mine/config.toml`.
- Read requirements (`REQUIREMENTS.md`) and current implementation evidence before changing design or code.

## Design rules

- Separate current implementation, accepted target design, assumptions, local decisions, and unresolved material decisions.
- Do not invent data fields, API behavior, tool names, command flags, or external semantics. Cite repository evidence or mark uncertainty.
- Verify external behavior against opened official or primary documentation and link the exact source in design and plans.
- Follow SOLID at real change boundaries. Do not create speculative interfaces, factories, plugin systems, or indirection without a demonstrated variation or testing boundary.
- Update design before creating a plan that depends on changed design.
- Material architecture decisions require repository-owner approval through the
  ADR lifecycle before their scope is planning-ready. Agents never infer or
  fabricate ADR approval; approved decisions are superseded, not reversed in place.

## No historical baggage by default

This is a new project unless the user explicitly states otherwise. When a later accepted design conflicts with an earlier internal implementation, change the target implementation directly. Do not keep reserved fields, obsolete parameters, compatibility aliases, dead interfaces, transitional adapters, duplicate schemas, or shims solely to preserve an abandoned plan. If cleanup is too large, create an explicit follow-up plan to remove the obsolete design rather than silently retaining technical debt.

## Document boundaries

- `AGENTS.md` contains durable repository-wide agreements only.
- Package-specific schemas, contracts, implementation decisions, and plan details belong in the design knowledge base and the relevant plan.
- Do not maintain an accumulating plan index here.

## Plan immutability

- A plan becomes immutable when handed to an implementation agent, execution begins, or a report records execution.
- Do not edit, rename, renumber, delete, or replace an immutable plan.
- Correct design first, then create a next-numbered compensating plan.
- If execution proves an immutable Plan infeasible, stop without claiming
  `IMPLEMENTED`; after independent evidence and owner-approved Design, reject
  it through the evidence-bound CLI transition and continue only through its
  next sibling compensation. Never hand-edit graph state to escape
  `IN_PROGRESS`.
- Plan IDs follow `planNN(?:-NN)*(?:-CNN)?`; files are
  `docs/plan/<id>-<slug>.md`. Compensation uses the next sibling `C` ordinal
  and reuses the base lineage worktree.

## Plan execution

- Read and understand the whole requested plan, governing design sections, predecessors, and registered official sources before editing.
- Fetch registered sources; do not implement from the planner's paraphrase alone.
- Ask the user when an uncovered decision changes product behavior, architecture, persistence, public contracts, security/privacy, compatibility, deployment responsibility, or acceptance criteria.
- Resolve bounded local implementation decisions using design, official documentation, and repository convention; record them in the implementation report.
- Preserve unrelated changes and stage only explicit in-scope files.
- Implementation agents may conclude `IMPLEMENTED`, never `ACCEPTED`. Independent review is required for `ACCEPTED`.

## Lightweight maintenance

MINE governs engineering change, not every repository edit. Direct, bounded, behavior-preserving maintenance may proceed without a MINE execution Plan when the correct result is unambiguous and no durable engineering contract changes: typo fixes, prose cleanup, translation, broken-link repair, README improvements, user-facing documentation additions, formatting-only documentation changes, examples that merely describe already-accepted behavior, and comments that do not change or establish behavioral contracts.

Such work still must: respect this `AGENTS.md`; preserve unrelated changes; stage explicit files; run relevant validation; use normal commit discipline; and never silently change product behavior while claiming to be docs-only.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [6ixGODD/mine](https://github.com/6ixGODD/mine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
