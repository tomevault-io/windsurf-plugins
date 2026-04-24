---
trigger: always_on
description: This file explains how to read and maintain this repository inside Codex.
---

# Meta_Kim for Codex

This file explains how to read and maintain this repository inside Codex.

## Human Summary

If you only remember four things:

- Meta_Kim is one cross-runtime governance system, not a pile of unrelated prompt folders (Claude Code, Codex, OpenClaw, and Cursor are projections of the same layer).
- `meta-warden` is the default public front door.
- `meta-conductor` orchestrates rhythm; the execution stack is grounded in `meta-genesis`, `meta-artisan`, `meta-scout`, `meta-sentinel`, and `meta-librarian` (capability-first dispatch still beats hardcoded names).
- Long-term edits belong in `canonical/` and `config/contracts/workflow-contract.json`; runtime-facing trees under `.codex/`, `.agents/`, `.claude/`, `openclaw/`, and `.cursor/` are generated or mirrored — sync instead of hand-forking.

## Read This Repository Correctly

Do not interpret this repository as “a folder full of unrelated agent prompts”.

Interpret it as:

**one intent-amplification architecture, governed through meta units, projected into Claude Code, Codex, OpenClaw, and Cursor.**

## What “Meta” Means

In Meta_Kim:

**meta = the smallest governable unit that exists to support intent amplification**

A valid meta unit should:

- own one clear class of responsibility
- define what it refuses, not only what it does
- be reviewable on its own
- be replaceable
- be safe to roll back

## What Codex Is Looking At

When this repository is opened in Codex:

- `AGENTS.md` is the project guide you are reading now
- `.codex/agents/*.toml` contains the 8 Codex custom-agent mirrors
- `.agents/skills/meta-theory/` is the project skill mirror (directory layout); `.codex/skills/` may hold the portable skill surface depending on sync scope
- `codex/config.toml.example` is generated from `canonical/runtime-assets/codex/config.toml.example` and shows how user-global Codex can wire MCP and skills

**Cursor parity (same repo, fourth runtime):** `.cursor/agents/*.md`, `.cursor/skills/meta-theory/`, `.cursor/mcp.json` — all refreshed by `npm run sync:runtimes` per `config/sync.json`.

Important maintenance rule:

- `canonical/agents/*.md` and `canonical/skills/meta-theory/SKILL.md` are the canonical sources
- `config/contracts/workflow-contract.json` is the canonical run-discipline and gate contract (not overwritten by agent/skill sync)
- `.codex/agents/*`, `.codex/skills/*`, `.agents/skills/*`, and Cursor/OpenClaw projection trees are derived runtime assets unless explicitly stated otherwise

## Capability-First Rule

Meta_Kim’s orchestration model is capability-first, not name-first.

That means:

- do not hardcode “call agent X” as the primary design rule
- first describe the capability needed
- then search for who declares ownership of that capability
- then dispatch the best match

The intended pattern is:

```text
Need capability X
-> Search agents / skills / capability index
-> Match by ownership boundary
-> Dispatch the best fit
```

Hardcoding a specific agent name without a search step is a design shortcut, not the canonical method.

## Default Behavior In Codex

The intended default behavior is:

1. the user gives raw intent
2. the system clarifies the intent first
3. the system searches for existing capabilities
4. the system decides whether specialist meta agents are needed
5. the system returns one coherent result

That is why the normal public front door should be:

- `meta-warden`

The other seven meta agents are backstage specialists, not the public menu.

## Critical Rule: Orchestrate Before You Execute

For complex development work, Codex should behave as an orchestrator first.

This applies to **all meta-theory Type flows**, not just development tasks:

- **Type A (Analysis)**: meta-theory gathers information, then dispatches via Fetch-first capability matching (quality audit → capability="code quality review" / synthesis → capability="coordination and synthesis")
- **Type B (Agent Creation)**: meta-theory plans, then dispatches via capability matching (identity → capability="agent SOUL design" / loadout → capability="skill/tool matching") for design work
- **Type C (Development)**: meta-theory handles Stages 1-3 with mandatory 3-STEP capability discovery, then dispatches via capability-matched `Agent` tool for Stages 4-8
- **Type D (Review)**: meta-theory reads the proposal, then dispatches via Fetch-first capability matching (quality audit → capability="code quality review" / external claims → capability="external capability discovery" / synthesis → capability="coordination and synthesis")
- **Type E (Rhythm)**: meta-theory diagnoses issues, then dispatches via capability matching (card deck → capability="workflow sequencing and rhythm control" / synthesis → capability="coordination and synthesis")

The core principle is: **meta-theory thinks, agents do.**

Treat these as complex tasks:

- multi-file work
- cross-module changes
- tasks requiring multiple capabilities or roles

For those tasks:

1. `Critical`: clarify the real request
2. `Fetch`: search for existing agents, skills, and tools
3. `Thinking`: define ownership, deliverables, sequencing, and boundaries
4. `Execution`: delegate using Codex-native custom agents or subagents
5. `Review`: inspect outputs against quality and boundary rules

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KimYx0207/Meta_Kim](https://github.com/KimYx0207/Meta_Kim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
