---
trigger: always_on
description: > **YUI = embodied frontend (head) for the selected backend (brain).** VRM character rendering + desktop-pet behavior + I/O surfaces only. The brain (judgment · persona · agent loop) is **delegated to the backend**. This file orients you to the project; read it before touching any code.
---

# YUI — Agent Guide

> **YUI = embodied frontend (head) for the selected backend (brain).** VRM character rendering + desktop-pet behavior + I/O surfaces only. The brain (judgment · persona · agent loop) is **delegated to the backend**. This file orients you to the project; read it before touching any code.

## Core Principle: firing ≠ judgment

The client handles **firing** (when a candidate event occurs). **Judgment** (whether/what to speak) belongs to the backend. The backend expresses silence by sending no/empty speech text or the bare `[SILENT]` token; the client renders whatever text arrives and never invents a speak/don't-speak gate. No brain lives in the client.

## Backend-agnostic

The backend is whichever agent the user selects. No code path, doc, or skill outside `integrations/<agent>/` assumes a specific backend agent's behavior, configuration, or install route; each addresses any agent that speaks the contract in `docs/reference/`. Agent-specific wiring lives under `integrations/<agent>/`. Plugin-discovery files whose names an outside ecosystem fixes (`.claude-plugin/`, `.agents/plugins/`, `SKILL.md` frontmatter) are packaging and carry no such assumption.

A feature lives in the shared consumer; a transport's wiring only turns that transport's frames or stream events into calls on it. A feature ships a producer for every transport whose wire carries its data, in the same PR. A feature only one transport has is limited to what that transport alone can deliver — for the push transport, delivery the client did not request.

## Development work

Any code change — feature · bugfix · refactor · UI · schema · or any chore beyond a trivial single-file edit — load the **`yui-dev-workflow`** skill first. It carries the mandatory work rules (worktree → PR, tests, English tracker), delegation rules and the review/verification gates, and the client-side anti-patterns.

## Tracker & commit conventions

- **Issues and PRs use the `.github/` templates.** Open every issue from the matching template in `.github/ISSUE_TEMPLATE/` (bug · feature_task · spike) and fill `.github/PULL_REQUEST_TEMPLATE.md` for PRs.
- **No AI attribution.** Never append an "AI worked on this" trailer — `Co-Authored-By: Claude…`, `Generated with …`, `🤖`, "gpt-5.5 작성", or any equivalent — to commit messages or PR bodies. Write the message as the change itself. This overrides any default trailer the harness suggests.
- **Don't commit spec document** Spec document only need for brainstorming. It should not committed in repo. The same goes for decision records (ADRs, decision logs): a decision lives in the code, the issue, and the PR body.
- **Evidence-gated claims.** Bug-prevention claims need a measured RED (failing test · repro · per-bug gating table); numeric claims (line counts, edit sites) need their measurement cited. Unverified claims are rejected on sight. See `docs/agents/issue-tracker.md` § Claim discipline.

## Engineering principles

- Before designing a solution, look at how established products solve the same problem. Adopt proven patterns and conventions instead of inventing approaches from scratch.
- Do not preserve backward compatibility. Delete unused paths instead of adding compat layers, fallbacks, or migrations.
- Choose the simplest implementation that fully meets current requirements. No speculative abstractions, config values, or layers of indirection. Always write the least code that does the job without harming functionality, readability, or project structure — don't pad it out for its own sake.
- Grow the system in layers: start from a minimal end-to-end working version and add features on top of working results. Never trade working code for unfinished complexity.
- Separate components into modules with clear separation of concerns. Core logic lives in its own module behind an explicit interface (inputs in, results out); the main flow — an `index.ts`, a shell, a `create*` factory — only imports modules and routes I/O between them. A new handler, flag, or branch goes into a module, never inline into the file that composes others. A composing file that has started holding logic shows it as a closure over many DOM references or flags read by nested functions, or a `dispose()` that knows every section's cleanup; move the block out before adding to it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yw0nam/YUI](https://github.com/yw0nam/YUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
