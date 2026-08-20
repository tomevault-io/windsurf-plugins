---
trigger: always_on
description: Instructions for AI coding assistants and developers working on the hermes-agent codebase.
---

# Hermes Agent - Development Guide

Instructions for AI coding assistants and developers working on the hermes-agent codebase.

## When the user asks to run Korgo Bot

Korgo Bot is the source-first desktop product in this repository. For setup or launch requests:

1. Do not edit product source just to install it.
2. Do not delete, reset, or overwrite `~/.hermes`; it may contain profiles, credentials, and conversation history.
3. Confirm the Mac has Git, Node.js 22.22+, npm, and `uv`. Explain any missing prerequisite before installing it.
4. Run `./scripts/setup-hermes-bots.sh --verify` from the repository root.
5. Launch with `npm --workspace apps/desktop run dev:bot`.
6. Preserve and report complete startup errors with credentials redacted.
7. Do not run generic `hermes update`; Korgo Bot source revisions and the compatibility-pinned remote template move together.

Follow [`docs/agent-assisted-setup.md`](docs/agent-assisted-setup.md) for the user-facing procedure.

**Never give up on the right solution.**

## What Hermes Is

Hermes is a personal AI agent that runs the same agent core across a CLI, a
messaging gateway (Telegram, Discord, Slack, and ~20 other platforms), a TUI,
and an Electron desktop app. It learns across sessions (memory + skills),
delegates to subagents, runs scheduled jobs, and drives a real terminal and
browser. It is extended primarily through **plugins and skills**, not by
growing the core.

Two properties shape almost every design decision and are the lens for
reviewing any change:

- **Per-conversation prompt caching is sacred.** A long-lived conversation
  reuses a cached prefix every turn. Anything that mutates past context,
  swaps toolsets, or rebuilds the system prompt mid-conversation invalidates
  that cache and multiplies the user's cost. We do not do it (the one
  exception is context compression).
- **The core is a narrow waist; capability lives at the edges.** Every model
  tool we add is sent on every API call, so the bar for a new *core* tool is
  high. Most new capability should arrive as a CLI command + skill, a
  service-gated tool, or a plugin — not as core surface.

## Contribution Rubric — What We Want / What We Don't

This is the project's intent layer. Use it two ways:

1. **For humans and for your own work** — what gets merged and what gets
   rejected, so a contribution aims at the target.
2. **For automated review (the triage sweeper)** — guidance on when a PR is
   safe to close on the three allowed reasons (`implemented_on_main`,
   `cannot_reproduce`, `incoherent`) and, just as important, **when NOT to
   close** one. Taste-based "we don't want this / out of scope" closes are NOT
   an automated decision — those stay with a human maintainer. The sweeper's
   job here is to recognize design intent and *avoid wrongly closing a
   legitimate contribution*, not to make the won't-implement call itself.

Read the balance right: Hermes ships a **lot** — most merges are bug fixes to
real reported behavior, and the product surface (platforms, channels,
providers, models, desktop/TUI features) expands aggressively and on purpose.
The restraint below is aimed squarely at the **core agent + the model tool
schema**, the one place where every addition is paid for on every API call.
"Smallest footprint" governs *how a capability is wired into the core*, NOT
whether the product is allowed to grow. We are expansive at the edges and
conservative at the waist.

### What we want

- **Fix real bugs, well.** The bulk of what lands is `fix(...)` against an
  actual reported symptom. A good fix reproduces the symptom on current
  `main`, points to the exact line where it manifests, and fixes the whole bug
  class — sibling call paths included — not just the one site the reporter hit.
- **Expand reach at the edges.** New platform adapters, channels, providers,
  models, and desktop/TUI/dashboard features are welcome and land routinely,
  including large ones (a new messaging channel, a session-cap feature, a
  Windows PTY bridge). Breadth in the product is a goal, not a footprint
  concern — as long as it integrates with the existing setup/config UX
  (`hermes tools`, `hermes setup`, auto-install) rather than bolting on a raw
  env var.
- **Refactor god-files into clean modules.** Extracting a multi-thousand-line
  cluster out of `cli.py` / `run_agent.py` / `gateway/run.py` into a focused
  mixin or module is wanted work, even when the diff is huge and mechanical
  (large `+N/-N` refactors merge regularly). The "every line traces to the
  request" test applies to *feature* PRs; a declared refactor's request IS the
  extraction.
- **Keep the core narrow.** New *model tools* are the expensive exception —
  every tool ships on every API call. Prefer, in order: extend existing code →
  CLI command + skill → service-gated tool (`check_fn`) → plugin → MCP server
  in the catalog → new core tool (last resort). See "The Footprint Ladder."
- **Extend, don't duplicate.** Before adding a module/manager/hook, check
  whether existing infrastructure already covers the use case. When several PRs
  integrate the same *category*, design one shared interface instead of merging
  them one at a time (see the ABC + orchestrator note under the Footprint

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nickvasilescu/korgo-bot](https://github.com/nickvasilescu/korgo-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
