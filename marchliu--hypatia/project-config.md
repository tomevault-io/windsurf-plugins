---
trigger: always_on
description: > This file configures Claude Code for Wheat research sprints. It is auto-maintained by `wheat init`. Edit with care.
---

# Wheat — Research Sprint

> This file configures Claude Code for Wheat research sprints. It is auto-maintained by `wheat init`. Edit with care.

## Sprint

**Question:** What is the architecture and design of Hypatia? Analyze its module structure, data flow, CLI design, storage patterns, and key design decisions.

**Audience:** engineers

**Constraints:**
- Rust codebase
- CLI tool for knowledge management

**Done looks like:** A comprehensive architecture overview with component descriptions, data flow diagrams, and design rationale

## Connectors

_No connectors configured. Use `/connect` to link org tools._

## Intent Router

When the user sends a plain message (no slash command), assess whether a Wheat command applies. If it does, announce which command you're running and execute it. If no command fits, respond normally.

**Route by intent:**

| User says something like...                                              | Route to                   | Why                         |
| ------------------------------------------------------------------------ | -------------------------- | --------------------------- |
| "look into X", "what about X", "explore X", "how does X work"            | `/research X`              | Information gathering       |
| "build X", "try X", "make a quick X", "test whether X"                   | `/prototype`               | Hands-on validation         |
| "is p001 really true?", "I doubt X", "what if X is wrong"                | `/challenge <id>`          | Adversarial testing         |
| "check this: <url>", "does <url> support X", "verify X against <url>"    | `/witness <id> <url>`      | External corroboration      |
| "what are we missing", "any gaps?", "what haven't we considered"         | `/blind-spot`              | Structural gap analysis     |
| "where are we", "what's the status", "show me the dashboard"             | `/status`                  | Sprint snapshot             |
| "write it up", "give me the recommendation", "summarize for the team"    | `/brief`                   | Decision document           |
| "make slides", "prepare for the meeting"                                 | `/present`                 | Stakeholder presentation    |
| "someone else is taking over", "hand this off", "document for successor" | `/handoff`                 | Knowledge transfer          |
| "combine with the other sprint", "merge these"                           | `/merge <path>`            | Cross-sprint merge          |
| "how did we get here", "show the history", "what changed over time"      | `/replay`                  | Sprint archaeology          |
| "we shipped, here's what happened", "actual results were X"              | `/calibrate --outcome "X"` | Prediction scoring          |
| "the stakeholder said X", "new constraint: X", "change of direction"     | `/feedback`                | Stakeholder input           |
| "resolve the conflict", "pick between X and Y"                           | `/resolve`                 | Conflict adjudication       |
| "connect to <repo/jira/docs>"                                            | `/connect <type> <target>` | External source linking     |
| "publish to confluence", "push to wiki", "sync to slack"                 | `/sync <target>`           | Artifact publishing         |
| "pull from deepwiki", "import from confluence", "backfill from repo"     | `/pull <source>`           | External knowledge backfill |

**When NOT to route:** Questions about the framework itself ("how does the compiler work?"), code edits to wheat files, general conversation, ambiguous intent. When in doubt, ask: "That sounds like it could be a `/research` -- want me to run it as a full research pass, or just answer the question?"

**Announce the routing:** Always tell the user what you're doing:

> Running as `/research "SSE scalability"` -- this will create claims and compile. Say "just answer" if you wanted a quick response instead.

This gives the user a chance to redirect before the full pipeline runs.

### Claims System (Bran IR)

- All findings are tracked as typed claims in `claims.json`
- Every slash command that produces findings MUST append claims
- Every slash command that produces output artifacts MUST run `wheat compile` first
- Output artifacts consume `compilation.json`, never `claims.json` directly
- The compiler is the enforcement layer -- if it says blocked, no artifact gets produced

### Claim Types

- `constraint` -- hard requirements, non-negotiable boundaries
- `factual` -- verifiable statements about the world
- `estimate` -- projections, approximations, ranges
- `risk` -- potential failure modes, concerns
- `recommendation` -- proposed courses of action
- `feedback` -- stakeholder input, opinions, direction changes

### Evidence Tiers (lowest to highest)

1. `stated` -- stakeholder said it, no verification
2. `web` -- found online, not independently verified
3. `documented` -- in source code, official docs, or ADRs
4. `tested` -- verified via prototype or benchmark
5. `production` -- measured from live production systems

### Claim ID Prefixes

- `d###` -- define phase (from /init)
- `r###` -- research phase (from /research)
- `p###` -- prototype phase (from /prototype)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MarchLiu/hypatia](https://github.com/MarchLiu/hypatia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
