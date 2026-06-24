---
trigger: always_on
description: Manages existing competitor files and orchestrates new discovery. The skill (Layer 1) handles:
---

# Startup Advisor — Local-First System Context

## What this is

A local-first startup advisor that guides founders through idea validation using Claude Code skills, reference files, subagents, and hooks — with all state stored as files under a `startup/` folder in the project root. No backend, no database, no UI. The agent is the workflow engine.

**Design principles:**
- Skills guide, subagents execute heavy work, files are the handoff
- All state lives in `startup/`
- Skills load only when relevant (progressive disclosure)
- One question at a time in all conversational flows
- Hooks nudge on convention violations but never block writes
- This is distributed as a Claude Code plugin. The repo root *is* the plugin (with `.claude-plugin/plugin.json` as the manifest and `.claude-plugin/marketplace.json` for the single-plugin marketplace). Skills, agents, and hooks live directly at repo root. Founder-facing runtime state (`startup/`) lives in the target project's root.

---

## Knowledge architecture

The system uses three layers of progressive disclosure:

| Layer | Location | When loaded | Purpose |
|---|---|---|---|
| **Layer 0** | `skills/using-startup-superpowers/SKILL.md` | When any startup task is in scope (description match) | System overview, artifact formats, file conventions, voice-input handling, subagent dispatch rules |
| **Layer 1** | `skills/*/SKILL.md` | When skill description matches | How to work with an artifact type — file conventions, status management, routing to Layer 2 workflows |
| **Layer 2** | `skills/*/references/*.md` | Explicitly loaded by Layer 1 | Step-by-step workflow for a specific scenario (e.g., first-time generation, initial discovery) |

Layer 0 describes **what** things are and loads via skill activation — no project files are injected. Layer 1 describes **how** to work with them. Layer 2 describes **how to run a specific workflow** end-to-end.

---

## Repository layout

The repo root is the plugin. The target project — where a founder actually works — only ever contains the `startup/` state directory; the plugin itself is loaded by Claude Code from wherever it's installed.

**Plugin (this repo):**
```
./
├── .claude-plugin/
│   ├── plugin.json                       # Plugin manifest (name, version, skills entrypoint, hook registrations)
│   └── marketplace.json                  # Marketplace listing (single-plugin marketplace, source: ./)
├── agents/
│   ├── web-researcher.md                 # Generic research subagent definition
│   ├── lean-startup-advisor.md           # Bias-isolated project assessment agent
│   ├── interview-analyst.md              # Bias-isolated interview transcript analysis agent
│   └── hypotheses-manager.md             # Bias-isolated hypothesis state assessment agent
├── hooks/                                # Hook scripts + hooks.json (Claude Code) and hooks-cursor.json (Cursor)
│   ├── auto-approve-startup.mjs          # PreToolUse: pre-approves Read/Write/Edit on paths under startup/
│   ├── validate-core-md.mjs              # PostToolUse: checks startup/core.md conventions
│   ├── validate-competitors-md.mjs       # PostToolUse: checks competitor .md file conventions
│   ├── validate-hypotheses-md.mjs        # PostToolUse: checks hypothesis .md file conventions
│   ├── validate-interview-scripts-md.mjs # PostToolUse: checks interview script .md file conventions
│   ├── validate-interview-md.mjs         # PostToolUse: checks interview analysis .md file conventions
│   ├── validate-mvp-plan-md.mjs          # PostToolUse: checks startup/mvp-plan.md conventions
│   └── validate-surveys-md.mjs           # PostToolUse: checks survey .md file conventions
└── skills/
    ├── using-startup-superpowers/
    │   └── SKILL.md                      # Layer 0: always-on conventions (file formats, voice, subagent dispatch)
    ├── whats-next/
    │   ├── SKILL.md                      # Layer 1: project direction + plan management
    │   ├── scripts/
    │   │   └── init-project.ts          # Legacy scaffold script (no longer used — initialization.md writes files directly)
    │   └── references/
    │       ├── initialization.md         # Layer 2: first-time project setup (routes to with-progress.md for tiers 2/3)
    │       ├── with-progress.md         # Layer 2: materials-based onboarding for founders with existing progress
    │       ├── b2c-painkiller.md        # Layer 2: B2C idea elaboration
    │       ├── b2b-painkiller.md        # Layer 2: B2B idea elaboration
    │       └── pivot-impact.md          # Layer 2: post-pivot artifact walk-through
    ├── competitors/
    │   ├── SKILL.md                      # Competitor management skill (Layer 1)
    │   └── references/
    │       ├── discovery.md              # Discovery workflow — first-time and reassessment (Layer 2)
    │       ├── user-feedback.md          # User-feedback mining workflow (Layer 2)
    │       └── watch.md                  # Competitor watch — on-demand landscape refresh (Layer 2)
    ├── market-research/
    │   ├── SKILL.md                      # Market research skill (Layer 1)
    │   └── references/
    │       └── initial-market-research.md # First-time research workflow (Layer 2)
    ├── hypotheses/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SergeiGorbatiuk/startup-superpowers](https://github.com/SergeiGorbatiuk/startup-superpowers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
