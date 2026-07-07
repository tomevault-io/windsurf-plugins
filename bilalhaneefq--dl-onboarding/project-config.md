---
trigger: always_on
description: This repository follows the Disrupt Labs venture-building standards: **STD-002 Venture Knowledge Foundation (VKF)** for knowledge, **STD-001 Spec-Driven Development (SDD)** for change. The sections below tell Claude Code when and how to use each command.
---

# Project Conventions for Claude Code

This repository follows the Disrupt Labs venture-building standards: **STD-002 Venture Knowledge Foundation (VKF)** for knowledge, **STD-001 Spec-Driven Development (SDD)** for change. The sections below tell Claude Code when and how to use each command.

> **Note:** This is the onboarding template. As you customize it for your venture (filling in the constitution, shipping features), update this CLAUDE.md to reflect your project's specifics — tech stack, conventions, tracker integration, etc.

---

## Standards Stack

| Standard | Governs                                              | Skill                  |
|----------|------------------------------------------------------|------------------------|
| STD-002  | Knowledge (constitution, specs layout, freshness)    | `venture-foundation`   |
| STD-001  | Change (proposals, spec-deltas, tasks, archival)     | `disrupt-sdd`          |
| STD-003  | Venture metrics                                      | (adopted as-needed)    |

**STD-002 is a prerequisite for STD-001** — you need a constitution and a specs/features layout before you can manage changes against them. A fresh fork of this template should run `/vkf/init` first, fill in the Core constitution files (mission, pmf-thesis, principles), and only then start SDD cycles.

---

## Knowledge Operations (STD-002)

### Command Routing Table

When the user asks about or attempts something, consult this table and suggest the right command:

| User says / situation | Command | What it does |
|----------------------|---------|-------------|
| "Initialize" / "Set up VKF" | `/vkf/init` | Bootstrap STD-002 structure |
| "Draft the mission/personas/..." | `/vkf/constitution` | Interactive constitution drafting |
| "We need to change our positioning/mission/..." | `/vkf/amend` | Tiered amendment process (C0–C3) |
| "I have a doc/sheet/notes to add" | `/vkf/ingest` | Classify and place external content |
| "What's missing?" / "What don't we know?" | `/vkf/gaps` | Scan for knowledge gaps with AI-proposed answers |
| "Are our docs up to date?" | `/vkf/freshness` | Check freshness + source staleness |
| "Are we compliant?" / "Check everything" | `/vkf/validate` | Full STD-002 audit |
| "Research competitors/market/..." | `/vkf/research` | Market research (exa.ai) for constitution sections |
| "Here's a meeting recording/transcript" | `/vkf/transcript` | Extract insights, generate meeting brief |
| "Where did this info come from?" | `/vkf/audit --trace` | Trace any section back to its sources |
| "What are our goals this quarter?" | `/vkf/okrs` | View/update quarterly objectives |
| "What needs attention?" | `/vkf/workflow status` | Show document lifecycle and pending actions |
| Pasting content in chat without context | Suggest `/vkf/ingest --inline` | Route pasted content through proper ingestion |
| Editing constitution files directly | Suggest `/vkf/amend` | Ensure proper change governance |

### Before Modifying Knowledge Base Files

Before editing any file in the specs tree, evaluate this decision tree:

1. **Is this a constitution file?** (any `.md` in `specs/constitution/`)
   - YES → Is the file still in Draft state (has `[REQUIRED]` placeholders)?
     - YES → Use `/vkf/constitution` for initial drafting
     - NO → Use `/vkf/amend` — announce: "This is an active constitution section. I'll use the amendment process."
   - NO → Continue normally (feature specs follow SDD)

2. **Is the user providing external content?** (pasting text, sharing a file reference)
   - YES → Announce: "I'll route this through `/vkf/ingest` to classify and place it properly."

3. **Is the user sharing a meeting transcript or recording notes?**
   - YES → Announce: "I'll use `/vkf/transcript` to extract and classify insights."

### VKF — Always / Ask First / Never

**Always:**
- Log every ingestion and transcript extraction to `specs/ingestion-log.yaml`
- Announce the amendment tier (C0 / C1 / C2 / C3) before making constitution changes
- Track "we don't know" as explicit knowledge state, not absence
- Update `Last amended` / `Last reviewed` dates on every document change
- Update `.claude/state/vkf-state.yaml` after significant operations
- Follow commit conventions: `[ingest]`, `[gaps]`, `[constitution]`, `[transcript]`, `[okr]`, `[workflow]`, `[foundation]`, `[validate]`

**Ask first:**
- Applying gap resolution answers that change active constitution content (routes through `/vkf/amend`)
- Archiving OKR quarters
- Transitioning documents from Active to Archived
- Overwriting an existing constitution file that has been filled out
- Running exa.ai research that consumes API credits

**Never:**
- Never overwrite audit log entries (append-only)
- Never delete gap reports (they are historical records)
- Never bypass amendment tiers — even if the user says "just change it", announce the tier
- Never auto-resolve gaps without user review
- Never mark a `[REQUIRED]` section as complete without content
- Never fabricate market data — use `/vkf/research` or clearly label assumptions

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BilalHaneefQ/dl-onboarding](https://github.com/BilalHaneefQ/dl-onboarding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
