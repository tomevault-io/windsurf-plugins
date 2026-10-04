---
trigger: always_on
description: 1. Read `docs/architecture/overview.md`.
---

# BearAgent repository instructions

## Read before changing code

1. Read `docs/architecture/overview.md`.
2. Read `docs/project/roadmap.md` and `docs/specs/README.md` to identify the current milestone and Feature.
3. Read the accepted Feature Spec, its active Implementation Plan, and related ADRs.
4. Inspect the current code and tests; chat history is never a source of truth.

## Project tracking

- `P0`, `P1`, and later milestones live in `docs/project/roadmap.md`.
- Feature IDs are global and stable. Every Feature Spec must declare `milestone: P<n>`; do not encode the milestone into `F-NNNN` or rename a Feature when it moves.
- Feature Spec and ADR filenames must begin with their full IDs: `F-NNNN-*.md` and `ADR-NNNN-*.md`.
- Feature status lives in the Feature Spec. Step-level progress lives in `docs/plans/PLAN-F-NNNN-*.md`.
- ADR status records whether a decision is accepted, not whether its implementation is complete.
- Keep at most one Implementation Plan `active`; reconcile its claims with code and tests before continuing it.
- Do not add a second Feature registry. Spec Front Matter is the status source; indexes and site `sourceRefs` must
  agree with it and are checked by `scripts/check_governance.py`.

## Git branches

- Feature branches created by Codex must use `codex/F-NNNN-<short-slug>`, where `F-NNNN` exactly matches the related Feature Spec ID.
- Use a concise lowercase kebab-case slug that describes the branch scope, for example `codex/F-0003-sqlite-event-store`.
- Keep one primary Feature per branch. If a Feature moves to another milestone, keep its stable Feature ID in the branch name.
- For S0 fixes or documentation work with no Feature Spec, use `codex/fix-<short-slug>` or `codex/docs-<short-slug>`.

## Change classification

- S0, trivial repair: implementation plus targeted verification; add a regression test for executable defects, not
  for typo or formatting-only edits. Update public docs only when observable behavior changes.
- S1, feature or behavior change: accept a concise Feature Spec before implementation. Add a Plan only when the
  work needs multiple independently verifiable slices or cannot be reviewed safely as one coherent change.
- S2, cross-module architecture, security boundary, persistence schema, public contract, or new production
  dependency: accept a full Feature Spec, an ADR, and an active Plan; include explicit failure, recovery, migration,
  rollback, and security evidence where applicable.

BearAgent is a Complex repository, but that does not make every change S2. Do not create ceremonial documents for
formatting-only or mechanical changes.

## Documentation synchronization

- Engineering facts live in `docs/`, code, and tests. `site/` explains those facts to readers; it must not invent a second version of the system.
- Every Feature must assess four surfaces: authoritative `docs/`, the beginner learning path, developer
  documentation, and public current status. For each surface, name the changed path or record `N/A` with a concrete
  reason in the Spec. Assessment is mandatory; editing every surface is not.
- Update beginner or developer pages only when their reader-visible explanation changed. Update public status only
  when a usable capability, limitation, or milestone state changed. Internal refactors do not need ceremonial site
  edits.
- Closing every milestone `P<n>` must update the Roadmap plus the site learning map, developer architecture/status summary, and milestone outcome. Do this before selecting the next milestone.
- External material may explain concepts or provide comparisons, but it cannot establish BearAgent behavior. Prefer primary sources, including the AI Agents in Depth book, DeepTutor documentation, and official documentation for high-star or otherwise relevant Agent projects; verify each project's current maintenance status. Treat star count as a discovery signal, not proof of correctness, and record source links.
- Public pages must distinguish general concepts, accepted design, current implementation, and future plans. Never copy a reference project's capability into BearAgent's current-state claims.
- Treat questions raised while the project owner reads code as documentation feedback. Verify each answer against `docs/`, code, and tests; then fold the reusable explanation into the relevant beginner page with a minimal example, a diagram when it clarifies flow, and explicit current/planned boundaries. Do not publish chat transcripts. Update learning indexes and cross-links, and update public status only when an implementation claim changed.

## Documentation writing

- Start with the reader's concrete question or a short execution example. Introduce a term only after the reader can see what it names.
- Keep established terms such as Runtime, port, adapter, Event, reducer, schema, Provider, Tool, Run, and Activity when they are the precise name. At first use, explain their job in one ordinary sentence. Do not replace them with longer invented Chinese phrases.
- Describe observable behavior instead of abstract proof claims. For example, write: “The same tests run against the in-memory and SQLite stores. Callers therefore use both stores in the same way.” Do not write: “The contract suite proves that port semantics are adapter-independent.”

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CherryYang05/BearAgent](https://github.com/CherryYang05/BearAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
