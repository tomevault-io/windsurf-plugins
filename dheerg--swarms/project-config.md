---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Anti-ratchet constraint on the briefing templates

- **The briefing templates are FIXED** (canonical in skills/workflow-rules/SKILL.md). No added sections, no prescribed investigation steps, no "first action" items, no acknowledgment rituals. If a member needs more context, improve the noun-phrase identity in suggest-members — never the template. (Observed regression vector: f7db555 sprayed framing into every brief → premature-termination and step-repetition failures.)
- **Carve-out: harness protocol mechanics are permitted.** One instruction on HOW members communicate (SendMessage is the wire; plain text dies with the turn) is transport, not task prescription. Task framing, lifecycle framing, phase framing, and rituals remain forbidden.

## What This Is

Swarm is a Claude Code plugin for launching agent teams. Eight commands drive an interactive setup that creates a coordinated team with defined roles and rules; users extend it with custom mode skills — full modes or thin wrappers over a built-in mode.

## Architecture

**Everything is a prompt.** No runtime composition, no imports, no framework — self-contained markdown consumed in one pass.

```
commands/launch.md          # Catch-all command — interactive team setup (Steps 0–8)
commands/code.md            # Mode shortcut — pre-selects Code, delegates to launch.md
commands/triage.md          # Mode shortcut — pre-selects Triage, delegates to launch.md
commands/write.md           # Mode shortcut — pre-selects Writing, delegates to launch.md
commands/refine.md          # Standalone — runs Review/Refine/Deliver against the current branch + PR
commands/workflow.md         # Custom mode entry point — takes a mode skill name, delegates to launch.md
commands/create-workflow.md  # Scaffolding — interviews user, generates mode skill + shortcut command (wrapper or full)
commands/update-workflow.md  # Refresh — regenerates the plugin-owned wiring of an existing shortcut command
skills/code-mode/           # Code mode: lead identity, facilitator title, rules, phase arc
skills/triage-mode/         # Triage mode: diagnose an issue (cause + blast radius), no code change; phase arc has no Refine
skills/writing-mode/        # Writing mode: lead identity, facilitator title, ownership boundaries, editorial baseline, phase arc
skills/general-mode/        # General mode: silent fallback + wrapper base — no shortcut command
skills/workflow-rules/      # CANONICAL governance spec — hard rules, briefing templates, gate presentation contract, launch mechanics, pulse
skills/gate-presentation/   # Frozen per-gate constants (question/options/digest/preview) — invoked fresh at each gate
skills/suggest-members/     # Recommends team composition based on outcomes and mode
skills/writing-style/       # Structural pattern analysis (trope detection) for writing-mode review
skills/resolve-dispute/     # Resolves stuck review findings via put-up-or-concede exchange
skills/define-rubric/       # Available skill for teams that genuinely need formal validation criteria
skills/independent-review-loop/  # Independent pre-delivery review loop — Codex or swarm-native fallback
agents/swarm-member.md      # Read-only team-member agent definition — every spawned teammate
agents/swarm-reviewer.md    # Ephemeral read-only reviewer for the independent-review-loop fallback — never a teammate
.claude-plugin/plugin.json  # Plugin manifest
.claude-plugin/marketplace.json  # Marketplace registry entry
.claude/swarm-ship.md       # Per-project ship definition (created at first launch, user-owned)
```

- **Commands** spawn teams (Agent tool; teams form implicitly at first spawn; requires Claude Code ≥ v2.1.178). Shortcuts read launch.md via `${CLAUDE_PLUGIN_ROOT}` and run it with mode pre-set; `/swarm:workflow` takes a custom mode name as argument.
- **Skills** are Skill-tool helpers — they cannot launch teams. Mode skills are invoked by the lead at Step 8b and return the phase arc + mode rules. `swarm:workflow-rules` is the canonical governance spec (hard rules, briefing templates, gate presentation contract, launch mechanics, pulse) — invoked by launch.md Step 1 and by project-local commands that cannot read `${CLAUDE_PLUGIN_ROOT}`. `swarm:gate-presentation` holds the frozen per-gate constants, invoked fresh at each gate.

### How launch.md works

- **Flow:** Step 0 pre-flight → Step 1 governance → Step 2 (outcomes → explicit tier pick → silent mode inference + suggest-members → team approval) → Step 7 confirmation → Step 8 spawn. Steps 3–6 are definitions serving Step 2 and "I have changes," not a walked sequence.
- **Three gates stand between outcomes and spawn — tier, team, launch.** Inline `$ARGUMENTS` outcomes exempt none of them (that skip caused a real regression). `$ARGUMENTS` is substituted before the model sees the prompt; when present, only the outcomes question is skipped.
- **Step 7 and Step 8 labels are load-bearing cross-references** throughout the plugin — keep the names.

### Phase arc


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DheerG/swarms](https://github.com/DheerG/swarms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
