---
trigger: always_on
description: You are reviewing a pull request in **outl**, a local-first outliner with a CRDT-based tree sync engine, written in Rust.
---

# Copilot review instructions — outl

You are reviewing a pull request in **outl**, a local-first outliner with a CRDT-based tree sync engine, written in Rust.
Read this whole file before commenting.
Your job is **not** a style pass — fmt, clippy, and CI already enforce style.
Your job is the review a Staff/Principal engineer would give: catch correctness, architecture, and scalability problems that humans miss, and only speak when it matters.

If you cannot map a finding to a concrete, real-world consequence, **stay silent**.
Noise costs reviewer attention; a single sharp comment earns trust.

---

## 0. Read these first

- Root `CLAUDE.md` — project-wide invariants and conventions.
- The `CLAUDE.md` inside the crate(s) the PR touches (e.g.
  `crates/outl-core/CLAUDE.md`).
- `CONTRIBUTING.md` — the merge bar and "decisions you don't get to revisit".
- `docs/contributing.md` — the review policy this file mirrors.
- `docs/development.md` — the engineer onramp (build / run / test / debug / CI / release).
  Load it when the PR touches CI workflows, slash commands, hooks, agents, or anything else a contributor's first 30 minutes depend on.
- `docs/architecture.md`, `docs/crdt.md`, `docs/markdown-format.md` — load the relevant one when the PR touches that area.
- The PR description and any linked issue.

Treat the per-crate `CLAUDE.md` as authoritative over generic Rust opinions.
If your suggestion contradicts it, drop the suggestion.

---

---

## Where the rest of this lives

This file is deliberately short — GitHub recommends repository-wide instructions stay within about two pages, and everything below applies only to certain paths.
Copilot loads these **in addition** to this file whenever a changed file matches their `applyTo` glob, so a Rust PR gets the Rust bar without a Solid PR paying for it.

| File | `applyTo` | Carries |
|---|---|---|
| [`instructions/rust.instructions.md`](instructions/rust.instructions.md) | `**/*.rs` | Rust quality bar, hot-path performance rules, testing bar |
| [`instructions/shared-primitives.instructions.md`](instructions/shared-primitives.instructions.md) | `crates/**` | The shared primitives catalog — check before approving any new helper |
| [`instructions/architecture.instructions.md`](instructions/architecture.instructions.md) | `crates/**` | Reuse-first violations, documentation-drift blocking |
| [`instructions/markdown.instructions.md`](instructions/markdown.instructions.md) | `**/*.md` | Semantic line breaks, one owner per fact |

Two things worth knowing about that split.
There is **no include mechanism** — each file is loaded whole or not at all, so a rule that must always apply belongs in this file, not in a path-scoped one.
And the primitives catalog is a **mirror** of `docs/shared-primitives.md` + `docs/primitives-*.md`; the two sides must change together, which `.claude/hooks/catalog-sync-guard.sh` enforces.

## Why a change was made: read the RFC

`docs/rfcs/` records why each non-obvious decision was taken, what was rejected, and what the change made *worse*.
When a PR touches an invariant, a data format, the CRDT, sync, or a projection path, it should carry an RFC — see [`docs/rfcs/README.md`](../docs/rfcs/README.md).
A diff that contradicts its own RFC is a blocker, exactly like one that contradicts an invariant.

**If a PR changes behaviour an existing RFC pinned and does not update that RFC, say so.**
That is the regression this process exists to catch: [RFC 0210](../docs/rfcs/0210-md-content-outside-op-log.md) documents a fix that silently deleted user content for months because the mirrored case was never written down.

---

## 1. Gate the PR before reviewing code

**Before reading the diff**, evaluate the PR description:

- Is there a linked issue (`Closes #N`, `Fixes #N`, `Related to #N`)?
- Is the problem the PR solves stated in one paragraph, in plain language?
- For a refactor: is *why now* explicit?
  ("Code is cleaner" is not enough.
  Either it unblocks something concrete, or it pays down debt the description names.)
- For a fix: is the bug behaviour described, with repro or a failing test?
- For a feature: does it match an item on `docs/roadmap.md` or an approved issue?

**If the description fails this gate**, your first and only top-level comment should be:

> Before I can review this PR meaningfully, the description needs a linked issue or a concrete problem statement.
> What real user-facing problem does this solve, and why now?
> If this is exploratory, please mark it as a draft and add an `RFC` label.

Do not proceed to line-level comments until that is fixed.
Reviewing a diff without knowing what problem it solves produces opinions, not review.

**Exception:** typo fixes, doc-only changes under `docs/` or `README.md`, and dependency bumps with a clear changelog link can skip this gate.

---

## 2. Non-negotiable invariants

These are project-level invariants.
A PR that violates any of them is a **blocker**, regardless of how clean the code looks.
Quote the invariant by name in your comment.

1. **Op log is source of truth.**
   Mutations flow through `Op` → `apply_op` → log.
   The materialized tree and the `.md` files are projections.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [outlmd/outl](https://github.com/outlmd/outl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-08 -->
