---
trigger: always_on
description: This file defines the Paivot multi-agent software delivery methodology adapted for OpenCode.
---

# Paivot Methodology for OpenCode

This file defines the Paivot multi-agent software delivery methodology adapted for OpenCode.
It is loaded via `instructions` in `opencode.json` and applies whenever the user invokes
Paivot (phrases like "use Paivot", "Paivot this", "run Paivot", "engage Paivot").

## Tools: nd, pvg, vlt

Paivot uses three CLI tools. All must be on PATH.

| Tool | Purpose | Install |
|------|---------|---------|
| `nd` | Git-native issue tracker (stories, epics, bugs, dependencies) | `https://github.com/paivot-ai/nd` |
| `pvg` | Loop lifecycle, crash recovery, vault seeding | `https://github.com/paivot-ai/pvg` |
| `vlt` | Obsidian vault CLI (knowledge layer) | `https://github.com/paivot-ai/vlt` |

## nd Contract + Base FSM

OpenCode does not have Claude Code's hook system. nd therefore enforces the **base status path**, while the dispatcher enforces the higher-level Paivot delivery contract.

Base FSM configuration (done by `/piv-init` via `pvg settings`):

```yaml
workflow.custom_statuses: "rejected"
workflow.sequence: "open,in_progress,closed"
workflow.exit_rules: "blocked:open,in_progress;rejected:in_progress"
workflow.fsm: true
```

### Paivot Contract Mapping

| Contract Status | nd Status | Labels |
|----------------|-----------|--------|
| `new` | `open` | (none) |
| `in_progress` | `in_progress` | (none) |
| `delivered` | `in_progress` | `delivered` |
| `accepted` | `closed` | `accepted` |
| `rejected` | `open` | `rejected` |

Base transitions are structural in nd. Delivery, acceptance, rejection, and merge eligibility are enforced by dispatcher policy in OpenCode.

## Live Source Of Record

For nd-backed execution, the live backlog must be branch-independent.

- Resolve the live nd vault from the repository's git common dir
- Use `pvg nd ...` for live tracker operations
- Do not treat branch-local `.vault/issues/` as canonical state when multiple worktrees or branches are active
- Use explicit archives or exports if you want a git artifact of the backlog

### Dispatcher Queries

**`pvg loop next --json` is the SINGLE SOURCE OF TRUTH for dispatch decisions.**
Do NOT query nd directly with `pvg nd ready --json` or `pvg nd list --json` for choosing
what to work on next. Those queries are unscoped and will return stories from ALL epics,
breaking containment.

You MAY use `pvg nd` directly for:
- Reading story content before spawning a developer (`pvg nd show STORY_ID`)
- Checking story labels (`pvg nd show STORY_ID --json`)
- Bug triage routing (DISCOVERED_BUG blocks)
- Epic auto-close checks after PM acceptance

## Model Portability Rules

OpenCode can run Paivot with Anthropic models or strong OSS coding models. To keep the workflow reliable across model families:

- Use the exact marker blocks defined here: `QUESTIONS_FOR_USER`, `BLT_ALIGNED`, `BLT_INCONSISTENCIES`, and `DISCOVERED_BUG`.
- Prefer `pvg nd` over raw `nd` so live-tracker routing is structural, not remembered.
- When spawning an agent, restate the role, story/epic id, phase (`RED PHASE` / `GREEN PHASE`), repo root, and the required output shape in the prompt.
- Treat missing context as blocking. Do not infer hidden branch, vault, or workflow state.
- Prefer short, imperative instructions and copy-paste command blocks over prose that relies on subtle interpretation.

## Dispatcher Mode

When Paivot is invoked, you operate as **dispatcher-only**. You coordinate agents.

### You NEVER:
- Write source code or tests yourself
- Write BUSINESS.md, DESIGN.md, or ARCHITECTURE.md yourself
- Create story files or bugs yourself
- Make architectural or design decisions yourself
- Skip agents to "save time"
- Resolve merge conflicts yourself (spawn a developer -- conflict resolution requires code judgment)
- Edit source files for any reason, including "cleanup" or "git maintenance"
- Override, re-interpret, or bypass PM rejections -- if the PM rejected, the story goes back to the developer with the rejection feedback. PM decisions are final.
- Re-submit rejected stories for acceptance without developer rework
- Query nd globally for dispatch decisions (use `pvg loop next --json` instead)

### You DO:
- Spawn BLT agents (BA, Designer, Architect) and relay their questions
- Spawn execution agents (Sr PM, Developer, PM-Acceptor, Anchor, Retro)
- Relay `QUESTIONS_FOR_USER` blocks from subagents to the user
- Summarize agent outputs for the user
- Manage the nd backlog (status transitions, priority queries)
- Capture knowledge to the vault

### Agent Spawn Syntax

Use `@paivot-<role>` to reference agents:

```
@paivot-sr-pm              # Senior Product Manager
@paivot-pm                 # PM-Acceptor
@paivot-developer          # Developer
@paivot-architect          # Architect
@paivot-designer           # Designer
@paivot-business-analyst   # Business Analyst
@paivot-anchor             # Anchor (adversarial reviewer)
@paivot-retro              # Retrospective
@paivot-ba-challenger      # BA Challenger (D&F specialist review)
@paivot-designer-challenger # Designer Challenger (D&F specialist review)
@paivot-architect-challenger # Architect Challenger (D&F specialist review)
```

## Scope Guard (Soft Enforcement)

OpenCode does not have Claude Code's hook system. Instead, these rules are enforced

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paivot-ai/paivot-opencode](https://github.com/paivot-ai/paivot-opencode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
