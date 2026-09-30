---
trigger: always_on
description: Treat this entire repository as a green-field project until this rule is explicitly removed. No
---

# Silk agent instructions

## Green-field policy

Treat this entire repository as a green-field project until this rule is explicitly removed. No
current or historical API, behavior, data format, build artifact, or internal representation is a
compatibility contract.

- Always implement the clean design intended for the eventual stable release, and update every
  caller, test, fixture, and document in the same change.
- Delete superseded code. Do not retain compatibility shims, adapters, aliases, fallbacks,
  migrations, deprecated entry points, dual paths, or code selected only for old behavior.
- Do not introduce technical debt to stage a transition. A change is incomplete while the obsolete
  path remains or its cleanup is deferred to a future task.
- Prefer a breaking change over a compromised design. Existing usage inside this repository is a
  migration target to update, not a reason to preserve the old shape.

These conventions apply to the entire repository. The current `effect-patterns` skill is the
authoritative source for Effect architecture. When this file and that skill differ, follow the
skill and update this file rather than preserving an older convention.

## Papercuts

Maintain PAPERCUTS.md, a global log shared by all agents sessions of anything that slowed down development. When you lose time to one mid-session, append date · symptom · fix · project. Check this file first when tooling fails mysteriously.

## Agent skills

Follow [ATOM-REACT-STYLEGUIDE.md](ATOM-REACT-STYLEGUIDE.md) for Effect Atom and `@effect/atom-react` code.

Create or update an OpenSpec change only when Julia explicitly requests one. Do not infer that an
OpenSpec artifact is required from the kind, scope, or size of a task. The prescriptive language
definition and reference live in `apps/docs/content/reference/`.

## Repository workflow

- The workspace uses pnpm, Turbo, strict TypeScript, Oxfmt, Oxlint, and Vitest.
- Put public LLVM code in `packages/llvm/src` and tests in `packages/llvm/test`.
- Keep the public barrel at `packages/llvm/src/index.ts` explicit.
- Prefer opening a draft PR early, as soon as there is a coherent task-scoped commit to publish.
  Reuse the task's existing PR when available so CI starts while implementation and review
  continue.
- Run the cheapest relevant focused checks while implementing. Run a broader local command only
  when it gives change-specific evidence that CI does not provide, reproduces or diagnoses a CI
  failure, or Julia explicitly requests it. Do not rerun the complete CI-covered suite locally as
  handoff ceremony.
- Before the final push, finish every repository mutation, including generated artifacts and any
  implementation-task checkboxes in an explicitly requested OpenSpec change, then audit, commit,
  and push the intended head. Required
  pull-request CI on that exact head is the authoritative full-repository completion guard; wait
  for it to pass before handoff. If it fails, fix the cause, run affected focused checks, push the
  new head, and wait for that head's CI.
- CI, review, PR updates, and handoff are workflow gates, not implementation tasks. Do not add
  OpenSpec task-list items whose sole action is running or recording checks, obtaining approval,
  waiting for CI, or reporting the handoff. A passing gate never requires a follow-up repository
  edit or empty push. Report the exact failure and whether it predates the change.

## Minimal compiler privilege

- All source-callable compiler operations belong to the sealed `Intrinsic` namespace. Do not add a
  compiler-known standard-library actor or recognize a library declaration by spelling in semantic
  analysis, HIR, MIR, evaluation, or a backend.
- A new compiler feature exposes only the smallest target-neutral primitive needed to build its
  public API in ordinary Silk source. Keep validation, policy, generic selection, provider types,
  and safe reusable wrappers in the standard library.
- Use `service` for runtime-provided Effect contracts and lexical provider replacement. Use
  `interface` for compile-time conformance and specialization only; interfaces never create
  requirement rows, service slots, or runtime dispatch.

## Collaborative decision sessions

When a task requires a multi-decision interview, grilling session, or other branching design
process, publish a visible Codex task plan at the start. Show the major decision branches, mark
exactly one current branch in progress, and keep completed and pending branches visible so the user
can orient themselves. Treat the plan as a live map: update, split, reorder, add, or remove branches
as answers expose new dependencies or invalidate earlier assumptions.

Before every user-facing decision question in that session, also include a compact, friendly
Markdown checklist showing the overall session state. Use `✅` for completed branches, `🟡` for the
single current branch, and `⬜` for pending branches. Keep it pleasant and quickly scannable; group
items only when the map becomes unwieldy, and never omit the current branch or remaining work. Keep
this in-message checklist synchronized with the visible Codex task plan.

## One module per actor


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [julia-script/silk](https://github.com/julia-script/silk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
