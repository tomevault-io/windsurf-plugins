---
trigger: always_on
description: Instructions for AI coding assistants and developers working on the loki-agent codebase.
---

# Loki Agent - Development Guide

Instructions for AI coding assistants and developers working on the loki-agent codebase.
This root file holds only what applies everywhere. Each area has its own `AGENTS.md` (aim for
~8k chars; `agent/subdirectory_hints.py` delivers up to 32k and truncates head/tail with a warning
past that); see the **routing table** at the end and read the area file before editing in that area.

**Never give up on the right solution.**

## What Loki Is

Loki is a personal AI agent that runs the same agent core across a CLI, a messaging
gateway (Telegram, Discord, Slack, ~20 platforms), a TUI, and an Electron desktop app. It
learns across sessions (memory + skills), delegates to subagents, runs scheduled jobs, and
drives a real terminal and browser. It is extended primarily through **plugins and skills**,
not by growing the core.

Two invariants shape almost every design decision and are the lens for reviewing any change:

- **Per-conversation prompt caching is sacred.** A long-lived conversation reuses a cached
  prefix every turn. Anything that mutates past context, swaps toolsets, reloads memories, or
  rebuilds the system prompt mid-conversation invalidates that cache and multiplies the user's
  cost. We do not do it; the ONE exception is context compression. Slash commands that mutate
  system-prompt state (skills, tools, memory) must be **cache-aware**: default to deferred
  invalidation (takes effect next session) with an opt-in `--now` flag (`/skills install --now`
  is the canonical pattern).
- **The core is a narrow waist; capability lives at the edges.** Every model tool is sent on
  every API call, so the bar for a new *core* tool is high. New capability should arrive as a
  CLI command + skill, a service-gated tool, or a plugin — not as core surface.

## Contribution Rubric — What We Want / What We Don't

The project's intent layer. It serves humans aiming a contribution AND the automated triage
sweeper, which may only close on `implemented_on_main`, `cannot_reproduce`, or `incoherent`.
Taste-based "out of scope" closes are a human maintainer's call; the sweeper's job is to
recognize design intent and *avoid wrongly closing a legitimate contribution*.

Read the balance right: Loki ships a **lot**. Most merges are bug fixes to reported
behavior, and the product surface (platforms, providers, models, desktop/TUI features)
expands aggressively on purpose. The restraint below targets the **core agent + model tool
schema**, the one place where every addition is paid for on every API call. "Smallest
footprint" governs *how a capability is wired into the core*, not whether the product may
grow: expansive at the edges, conservative at the waist.

### What we want

- **Fix real bugs, well.** Reproduce the symptom on current `main`, point to the exact line
  where it manifests, and fix the whole bug class — sibling call paths included.
- **Expand reach at the edges.** New adapters, channels, providers, models, desktop/TUI/
  dashboard features land routinely, including large ones — as long as they integrate with
  the existing setup/config UX (`loki tools`, `loki setup`, auto-install) rather than
  bolting on a raw env var.
- **Refactor god-files into clean modules.** Huge mechanical `+N/-N` extraction PRs are
  wanted work. "Every line traces to the request" applies to *feature* PRs; a declared
  refactor's request IS the extraction.
- **Keep the core narrow.** Prefer, in order: extend existing code → CLI command + skill →
  service-gated tool (`check_fn`) → plugin → MCP server in the catalog → new core tool (last
  resort). See the Footprint Ladder.
- **Extend, don't duplicate.** Check whether existing infrastructure covers the use case
  before adding a module/manager/hook. When 3+ open PRs integrate the same *category*
  (memory backends, providers, notifiers), design an ABC + orchestrator, wrap the existing
  built-in as the first provider, and turn the competing PRs into plugins against it.
- **Behavior contracts over snapshots.** Tests assert how two pieces of data relate, never
  freeze a current value (see Testing).
- **E2E validation, not just green unit mocks.** Anything touching resolution chains, config
  propagation, security boundaries, remote backends, or file/network I/O must exercise the
  real path with real imports against a temp `LOKI_HOME`. Mocks hide integration bugs.
- **Cache-, alternation-, and invariant-safe.** Preserve prompt caching, strict role
  alternation (never two same-role messages in a row; never a synthetic user message injected
  mid-loop), and a system prompt byte-stable for the life of a conversation.
- **Contributor credit preserved.** Salvage external work by cherry-picking (rebase-merge) so
  authorship survives; build on top rather than reimplementing.

### What we don't want (rejected even when well-built)

- **Speculative infrastructure.** Hooks/callbacks/extension points with no concrete consumer.
  Adding a hook is easy; removing one after plugins depend on it is hard. A hook with a real,
  stated use case is NOT speculative even if the consumer ships separately.
- **New `LOKI_*` env vars for non-secret config.** `.env` is for secrets only. Behavioral

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wundercorp/loki](https://github.com/wundercorp/loki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
