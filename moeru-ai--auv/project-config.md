---
trigger: always_on
description: Concise but detailed reference for contributors working on AUV. Improve code
---

# AUV Agent Guide

Concise but detailed reference for contributors working on AUV. Improve code
when you touch it; avoid one-off patterns and keep the shared runtime model in
view.

## Project Mission

- AUV turns application UI workflows into command-like, inspectable,
  replayable, and eventually shortcut-like operations.
- AUV is not only a CLI wrapper and not a generic LLM agent.
- Design around reusable runtime APIs, first-party drivers, implicit run
  recording, artifact capture, replay, and inspection.
- Keep CLI, MCP, library calls, and future UI surfaces on the same execution
  model.
- Prefer explicit boundaries between runtime, drivers, Rust command
  frontends, run storage, and reference documentation.
- Use `docs/TERMS_AND_CONCEPTS.md` as the shared vocabulary for run recording,
  inspection, trace data, artifacts, and viewer-facing APIs.
- When a design introduces or changes a core term, update
  `docs/TERMS_AND_CONCEPTS.md` instead of defining the term only inside a
  transient spec.

> Many project details are still undecided. During design and implementation,
> communicate with users frequently and clearly to avoid misunderstandings,
> premature naming decisions, and avoidable rework.

## Project Phase: Restore The AUV Core Lane

AUV is currently pulling its active roadmap back to the Application Use Via
core: invoke, run recording, artifacts, inspection, app-local Rust commands,
and distill/compile/run reuse across frontends. The former SkillBundle surface
has been retired; do not reintroduce bundle execution, export, or verification
as compatibility. The important work is to make the remaining runtime surfaces
agree on one shared execution model. Prefer changes that tighten or reconnect
the existing core runtime over polishing one archived vertical proof.

The macOS AX copilot work remains valuable, but it is no longer the active
product lane. Treat `candidate-action` as a frozen archived vertical proof. Do
not present it as AUV itself, do not expand it with new action classes or
product polish, and do not route new roadmap work through TextEdit-only proof
paths.

Good convergence work usually has one of these shapes:

- Defines or tightens a shared contract in `docs/TERMS_AND_CONCEPTS.md`,
  `src/contract.rs`, run records, artifacts, or command signals.
- Reconnects invoke and typed Rust command surfaces so CLI, library, MCP, and future UI
  frontends share the same runtime execution path.
- Connects an existing producer to an existing consumer with typed evidence,
  for example `RecognitionResult -> CandidateRef -> action -> VerificationResult`.
- Aligns action-selection metadata with typed driver results, for example
  `ActionResolverDecision -> InputActionResult -> trace/artifact signals`.
- Fixes a reproduced bug in a narrow path and adds a regression test.
- Turns a known boundary into explicit metadata, failure layers, fallback
  reasons, or validation errors.
- Finishes an owner-approved slice without expanding it into adjacent roadmap
  work.

Poor convergence work looks like broad cleanup, TODO chasing, opportunistic
helper extraction, speculative backends, or implementing future APIs just
because a doc mentions them. It also includes continuing to deepen the archived
AX copilot vertical and then calling that AUV progress. If the change adds new
surface area, it needs an explicit reason tied to the current contract.

## Scope Discipline

Before editing, classify the change as one of: bug fix, test-only, docs-only,
narrow refactor, or approved feature. If none fits, ask for a smaller slice.

Some missing pieces are intentional deferrals, some are incomplete work, and
some are real bugs. Do not guess which one you are looking at. Classify the
gap from evidence, failing tests, owner instructions, or existing reference
docs before implementing anything.

Scope rules:

- Do not implement TODOs, roadmap notes, or future-phase designs unless the
  owner names that slice.
- Do not run broad repository scans and turn the findings into drive-by
  changes. Search only to understand the assigned slice.
- Do not mix unrelated cleanup into behavior changes. Mention cleanup
  opportunities as follow-up candidates instead.
- Cross-layer changes are allowed when they are the approved contract slice,
  but the dependency direction must be clear. Example: contract type -> driver
  artifact -> read-side inspector test.
- Avoid ad-hoc compatibility shims. Versioned read compatibility for existing
  run artifacts or public records is allowed when the migration
  boundary and tests are explicit.

Owner approval means the owner named the function/module/behavior, accepted a
concrete proposal, or asked for a specific next slice. A doc mentioning a
future feature, a TODO marker, or a change that feels like "the obvious next
thing" is not approval.

When a good idea appears mid-task, do not implement it inline. Record it as a
candidate next slice with one sentence explaining why it matters, then finish
the current slice.

When you decide *not* to implement something that a reader could plausibly
expect — a field, an enum variant, a branch, an algorithm stage, an API
surface, a fallback path, or any other surface that has been considered and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moeru-ai/auv](https://github.com/moeru-ai/auv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
