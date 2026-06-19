---
trigger: always_on
description: This document tells an agent **how to think about, structure, and incrementally
---

# SKILL — Authoring features in the saifctl repo

This document tells an agent **how to think about, structure, and incrementally
write a feature** under `saifctl/features/<name>/`. It captures the working
methodology used to author features like `_phases-and-critics` and
`release-readiness`. If you point a fresh agent at this file with the
prompt _"continue our feature work using SKILL.md"_, it should be able to
reproduce the workflow without further hand-holding.

The format follows saifctl's own filesystem-as-structure convention: every
file under a feature dir means something specific (see §3 below). The
_content_ of those files follows the conventions in §4–§7.

---

## 1. Reference examples

Two existing features that you can read end-to-end as templates:

- **`saifctl/features/_phases-and-critics/`**
- **`saifctl/features/release-readiness/`**

They look different because they describe different work, not because
they belong to different categories. Same workflow, same
`specification.md` structure, same ID conventions — both yield phases
when ready. Read one or both to see how the conventions in this
document play out in practice.

---

## 2. The workflow (in order)

Six phases, executed sequentially. Do not skip ahead — each phase's
output is the next phase's input.

### Phase A — Orient

Read what the user is actually asking. Identify:

- The **components** in scope (e.g. npm package, vscode-ext, web, docs,
  sandbox, the orchestrator loop, a specific subsystem, etc.).
- The **target state** they're aiming at (e.g. alpha release? new
  capability? migration? — this changes the bar).
- Which **pieces of the codebase / docs / config / prior conversations**
  you'll need to inspect to gather context.

Write it down. This is your reference point for the next phase. It also forces you to confront any ambiguity in the ask before you dive into the weeds.

### Phase B — Gather context (parallel exploration)

**Inventory first; design later.** You cannot write a useful spec until
you know the territory. Resist the urge to propose fixes / designs
during this phase — just record what's there.

Sources of context vary by feature. Common ones:

- **Code audit** — when the feature operates against existing code,
  delegate parallel sub-agents to walk the relevant subtrees.
- **README / docs / marketing copy** — when the feature involves
  matching claims against reality.
- **User conversations** — when the feature derives from a workflow
  the user has been doing manually. Capture verbatim quotes; they
  age into authoritative §0 Background in `specification.md` later.
- **External references** — issues, PRs, prior design docs, external
  tools the feature integrates with.

When delegating to parallel sub-agents, use `Explore`-type agents for
"find and inventory" tasks (read-only, context-cheap), and
`general-purpose` agents for "analyze and assess" tasks that need
judgment. Always:

- Brief each agent like a smart colleague who hasn't seen the
  conversation: _what to investigate, why it matters, what shape the
  report should take, how long_.
- Give each one a **distinct, non-overlapping scope** so you don't pay
  for duplicated work.
- Ask for a **structured report** with file:line citations. Pick the
  shape that matches your tracking needs (a flat punch list, a tagged
  table, a per-component list). Tell the agent to be specific —
  "tests fail" is useless; "vitest reports 3 failing in
  foo/bar.test.ts:42" is useful.
- Tell the agent **not to fix or design anything** during this phase.

Run sub-agents in parallel by issuing multiple `Agent` tool calls in
the same response. Synthesize the results yourself; do not delegate
the synthesis to another agent (that's where overclaiming sneaks in).

For the canonical example, see how the four parallel audits feeding
`release-readiness` are recorded in
`saifctl/features/release-readiness/specification.md` Appendix A.

### Phase C — First-pass `specification.md`

Create the feature dir at `saifctl/features/<name>/`. No underscore
prefix unless the feature is documentation-only / not runnable
(`_phases-and-critics`, `_phases-example` use the prefix because they
exist for reference, not for `feat run`).

Write `specification.md` first. Use the section structure in §4 below.
The first version is a **verbatim lift of the gathered context** —
every finding / requirement / user-stated constraint becomes a row in
§3 with a fresh ID. Don't try to be comprehensive; don't try to resolve
open questions; don't try to lock the phase breakdown. Capture, then
refine.

Note this in the file's preamble: _"This file is a working document.
The first pass is a verbatim lift of the gathered context. Subsequent
passes will refine."_ That sets expectations for the user reading it.

### Phase D — Iterative refinement (the conversation loop)

Walk the user through §6 (open questions). For each one:

1. Lay out the options briefly.
2. Capture the user's decision.
3. Migrate the question into §5 as `D-NN` with rationale + back-references
   (which work-item IDs it touches).
4. Update the affected work-item rows so they reference **Decision
   D-NN** (and update any tracking columns the spec uses to mark the
   row as resolved).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [safe-ai-factory/saifctl](https://github.com/safe-ai-factory/saifctl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
