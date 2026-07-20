---
trigger: always_on
description: Happy (2) is a desktop work and coding app that evolves by adopting itself. It is
---

# Agent Instructions

## Project

Happy (2) is a desktop work and coding app that evolves by adopting itself. It is
desktop-only: do not assume mobile use, add mobile-specific behavior, or adapt
layouts for mobile viewports.

## Feature development workflow

Treat each feature as one atomic, independently mergeable change, not as the
lifetime of a Conductor workspace. A worktree may contain only one unmerged
feature at a time. Do not begin the next feature until the current feature has
been pushed or merged to `main`; obtain review when its scope or risk warrants
it under the review workflow below.

After that merge, reuse the same workspace/worktree when convenient. It does
not need to be recreated, checked out directly on `main`, or have a branch tip
identical to `origin/main` before work starts. The next feature must remain a
separate, reviewable diff and must be rebased onto the latest `origin/main`
during the normal sync-to-main workflow. Create another Conductor workspace
only for parallel work or when another feature must begin while the current one
is still unmerged.

Build each feature in isolation, with an explicit boundary between its server
and UI work. Do not mix unrelated features into the same implementation.

## Review workflow

Review is not required for every edit. Reserve Claude Opus review for sizable
or critical changes: security or authorization behavior, durable data or
migrations, server API contracts, complex concurrency or synchronization,
substantial UI flows, or broad/high-risk diffs. Opus is deliberately slow, so
do not invoke it early, before the implementation and relevant tests are
complete, or for a small, isolated, low-risk change.

For a quick independent review, ask GPT Luna at high effort instead. Use that
option for focused, low-to-medium-risk diffs when a fast second look is useful.
Routine mechanical, small, and low-risk changes may rely on the implementer's
own verification and relevant automated checks without a separate review.

When an Opus review is warranted:

1. Finish the isolated implementation and its required tests, then review the
   complete task diff with Claude Opus at medium effort and streaming/verbose
   output. Do not use `ultrareview`/Ultracode for this gate.
2. Address every actionable finding, rerun the relevant checks, and resume the
   same persisted Opus session with a concrete account of the fixes.
3. Repeat until both Codex and Opus explicitly agree that no task-blocking issue
   remains. Never interrupt, terminate, cancel, or replace a running Claude
   reviewer merely because it is slow or has not produced intermediate output.
4. Run repository-wide `pnpm format`, then the final required checks and sync
   the task to `main` using the workflow below. Only after that merge may the
   same worktree begin the next feature.

For Claude-owned UI tasks that warrant review, Codex performs the reciprocal
review and Opus resumes the same session to address actionable findings. For
GPT-owned server tasks, Opus is a read-only reviewer and must not implement
server behavior.

Backward compatibility is not a default product requirement. Prefer the clean
new-server/backend and UI design unless the current task explicitly requires a
compatibility or data-preservation contract; do not add legacy branches solely
to preserve obsolete behavior.

## Model ownership

Model ownership is strict:

- GPT models, and only GPT models, implement the server behavior and its `gym`
  coverage.
- Claude Opus implements the UI portion only after the server behavior is
  complete and the user has explicitly approved the backend.

Development starts with the server feature. Design its API and data model
carefully, implement it, and prove its observable behavior with thorough `gym`
tests. Then stop and ask the user to review and approve the backend. Do not
begin or hand off any UI work until that approval is given. Favor simple,
durable boundaries that will not create foreseeable maintenance or
compatibility problems. Do not add abstractions, options, or behavior solely
for hypothetical future use cases; solve the feature currently being built
well.

## Design system

Before creating or changing any user interface, read and follow `DESIGN.md`.
It is the authoritative contract for component ownership, blueprint coverage,
layout dimensions, icon preparation, optical alignment, and cross-browser
rendering tests. Reusable visual components belong in `happy2-ui`; application
packages may only compose them and supply product state and event handlers.

Use flexbox for layout almost all of the time — it is the default for every row,
column, stack, toolbar, and centered box. Use another mechanism (CSS Grid, and
only for a genuine two-dimensional grid) solely when flexbox cannot express the
layout at all; never fall back to floats, `inline-block` hacks, or layout tables.
See `DESIGN.md` → "Layout with flexbox".

## Generated images

Whenever a feature needs a new raster image, generate an original image for
that feature. Never copy or reuse another feature's image as a placeholder.
Every new built-in plugin must include its own newly generated `plugin.png`
whose visual identity matches that plugin.

## Reactivity


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [slopus/happy2](https://github.com/slopus/happy2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
