---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Anti-ratchet constraint on launch.md

launch.md Step 8 briefing templates are FIXED. Do not add sections to member briefs. Do not prescribe investigation steps. Do not introduce "first action" items or acknowledgment rituals. If a team run reveals a member needs more context, the fix is to improve the noun-phrase identity in suggest-members, NOT to add sections to the briefing template. This constraint exists because the briefing templates are an observed regression vector — commit f7db555 sprayed "quality-oriented" framing into every brief and created FM-3.1 (premature termination) + FM-1.3 (step repetition) failure modes users observed.

**Carve-out: harness protocol mechanics are permitted.** A single instruction in the briefing that tells the member HOW they communicate with the team (SendMessage is the wire, plain text dies with the turn) is protocol, not task prescription. It does not describe what to investigate, when to act, or what "done" looks like — it only describes the transport layer. Protocol mechanics are allowed. **Task framing, lifecycle framing, phase framing, acknowledgment rituals, and "first action" directives remain forbidden.**

## What This Is

Swarm is a Claude Code plugin for launching agent teams. Eight commands — `/swarm:launch` (catch-all), `/swarm:code`, `/swarm:write`, `/swarm:general` (mode shortcuts), `/swarm:refine` (refine the current branch and PR), `/swarm:workflow` (custom mode entry point), `/swarm:create-workflow` (scaffolding), `/swarm:update-workflow` (refresh generated workflows) — drive an interactive setup that creates a coordinated team of agents with defined roles and rules. Users can extend swarm by creating custom mode skills in their own codebases — either as **full custom modes** or as **thin wrappers** that extend a built-in mode.

## Architecture

**Everything is a prompt.** No runtime composition, no imports, no framework. Commands and skills are self-contained markdown files consumed by the model in one pass.

```
commands/launch.md          # Catch-all command — interactive team setup (Steps 0–8)
commands/code.md            # Mode shortcut — pre-selects Code, delegates to launch.md
commands/write.md           # Mode shortcut — pre-selects Writing, delegates to launch.md
commands/general.md         # Mode shortcut — pre-selects General, delegates to launch.md
commands/refine.md          # Standalone — runs Review/Refine/Deliver against the current branch + PR
commands/workflow.md         # Custom mode entry point — takes a mode skill name, delegates to launch.md
commands/create-workflow.md  # Scaffolding — interviews user, generates mode skill + shortcut command (wrapper or full)
commands/update-workflow.md  # Refresh — regenerates the plugin-owned wiring of an existing shortcut command
skills/code-mode/           # Code mode: lead identity, facilitator title, rules, phase arc
skills/writing-mode/        # Writing mode: lead identity, facilitator title, ownership boundaries, editorial baseline, phase arc
skills/general-mode/        # General mode: lead identity, facilitator title, lightweight default
skills/workflow-rules/      # Governance spec for custom workflows — hard rules, briefing templates, launch mechanics
skills/refine-outcomes/     # Converts implementation descriptions into outcome statements
skills/suggest-members/     # Recommends team composition based on outcomes and mode
skills/writing-style/       # Structural pattern analysis (trope detection) for writing-mode review
skills/resolve-dispute/     # Resolves stuck review findings via put-up-or-concede exchange
skills/define-rubric/       # Available skill for teams that genuinely need formal validation criteria
.claude-plugin/plugin.json  # Plugin manifest
.claude-plugin/marketplace.json  # Marketplace registry entry
.claude/swarm-ship.md       # Per-project ship definition (created at first launch, user-owned)
```

**Commands** are entry points that can spawn teams (TeamCreate + Agent). Shortcut commands (`/swarm:code`, `/swarm:write`, `/swarm:general`) use `${CLAUDE_PLUGIN_ROOT}` to read launch.md and execute it with mode pre-set. `/swarm:workflow` is the generic entry point for custom modes — it takes a mode skill name as argument. `/swarm:create-workflow` scaffolds a custom mode skill + shortcut command in the user's project. **Skills** are helpers invoked via the Skill tool — they cannot launch teams. **Mode skills** (`swarm:code-mode`, `swarm:writing-mode`, `swarm:general-mode`, and user-defined custom modes) are invoked by the team lead at Step 8b; they return the phase arc and mode-specific rules for that run. `swarm:workflow-rules` returns the universal governance spec (hard rules, briefing templates, launch mechanics) for use by user-authored shortcut commands that cannot access `${CLAUDE_PLUGIN_ROOT}`.

### How launch.md Works

Step 0 (pre-flight) → Step 1 (universal hard rules) → Step 2 (outcomes + defaults/configure fork) → Step 3 (mode selection) → Step 4 (team members, mode-aware) → Step 5 (team shape) → Step 6 (lead research toggle) → Step 7 (confirmation) → Step 8 (spawn and execute).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DheerG/swarms](https://github.com/DheerG/swarms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
