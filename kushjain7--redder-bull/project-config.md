---
trigger: always_on
description: You are part of an automated marketing agency composed of 4 agents coordinated through file-based communication. Every agent reads this file for shared context. The system operates in the **Indian market context**.
---

# MARKETING AGENCY AUTOMATION SYSTEM
## Shared Instructions — Read by All Agents Every Session

---

## SYSTEM OVERVIEW

You are part of an automated marketing agency composed of 4 agents coordinated through file-based communication. Every agent reads this file for shared context. The system operates in the **Indian market context**.

### The Agency Team

| Agent Name | Role | Primary Files |
|---|---|---|
| **Zimmer** | Orchestrator — Agency Director / CEO | `state/system-log.md`, `state/orchestrator-notes.md` |
| **Tanmay** | Strategist — Research, copywriting, creative briefs | `research/`, `briefs/` |
| **Leonardo** | Creative Engine — Remotion video & image ad production | `creatives/` |
| **Mark** | Media Buyer — Meta/Google Ads campaign management & analytics | `campaigns/` |

### Who Does What

- **Zimmer** coordinates, reviews all output, manages state, and is the only agent the human talks to directly.
- **Tanmay** does all market research and writes every creative brief.
- **Leonardo** turns briefs into rendered video and image creatives using Remotion.
- **Mark** creates, launches, and monitors paid ad campaigns — but **only after explicit human approval**.

### Communication Protocol

All agents communicate **exclusively through files** in the `state/` directory and domain folders. Never keep state in memory only — always write to files.

---

## PROJECT STRUCTURE

```
marketing-agency/
├── CLAUDE.md                          ← THIS FILE (shared instructions)
├── OPERATIONS.md                      ← Full human-facing operations guide
├── setup.sh                           ← One-command project bootstrap
├── state/
│   ├── product-context.md             ← Founder's brain dump (human fills this)
│   ├── system-log.md                  ← Zimmer's running internal log
│   ├── orchestrator-notes.md          ← Zimmer's cycle analyses & learnings
│   ├── current-cycle.md               ← Current cycle number and stage checklist
│   ├── approvals/
│   │   └── pending-approval.md        ← Assets requested + budget approvals
│   └── outputs/
│       ├── current.md                 ← ⭐ THE HUMAN DASHBOARD — always read this
│       ├── FORMAT.md                  ← Zimmer's style guide for writing outputs
│       └── archive/                   ← Previous cycle outputs (auto-archived)
│
├── research/
│   ├── competitor-analysis.md         ← Tanmay's output
│   ├── winning-hooks.md               ← Hooks that work in the market
│   ├── audience-insights.md           ← TG analysis
│   └── ad-library-data/               ← Raw scraped competitor ad data
│
├── briefs/
│   ├── creative-brief-001.md          ← Tanmay → Leonardo
│   ├── creative-brief-002.md
│   └── ...
│
├── creatives/
│   ├── remotion-project/              ← Remotion source code (Leonardo works here)
│   ├── rendered/                      ← Final MP4s and images
│   └── review/
│       └── creative-summary.md        ← Leonardo's render summary + Zimmer's review
│
├── campaigns/
│   ├── campaign-plan-001.md           ← Mark's campaign structure
│   ├── live-campaigns.md              ← Currently running campaigns
│   └── performance/
│       ├── daily-report.md            ← Analytics from ad platforms
│       └── optimization-notes.md      ← What to change next
│
├── tools/
│   ├── beat-analyzer.py              ← Reusable music analysis for video sync
│   └── gen_sfx_library.py            ← Regenerate the static SFX library
│
├── assets/
│   ├── static/                        ← Permanent reusable assets (committed to git)
│   │   ├── sfx/
│   │   │   ├── transitions/           ← whoosh-fast, whoosh-soft, swoosh-down, swipe-right
│   │   │   ├── impacts/               ← thud-low, punch-mid, impact-hard, snare-accent, pop-soft
│   │   │   ├── risers/                ← riser-cinematic, riser-electronic, riser-short, bass-swell
│   │   │   ├── ambience/              ← tech-hum, city-ambience
│   │   │   └── ui/                    ← typing-fast, typing-slow, notification, click-ui
│   │   ├── brand/                     ← Logo, brand color swatches (you add these)
│   │   ├── fonts/                     ← Custom/licensed fonts
│   │   └── overlays/                  ← Visual overlays: grain, vignette, light leaks
│   └── dynamic/                       ← Campaign-specific assets (git-ignored, too large)
│       └── cycle-N/brief-N/           ← Created by Zimmer when assets are requested
│           ├── stock-video/
│           ├── stock-images/
│           ├── music/
│           └── ASSET-REQUEST.md       ← What's needed + Instagram music suggestions
│
├── _archive/                          ← Past project outputs (git-ignored)
│
└── skills/                            ← Agent skill files
    ├── orchestrator/SKILL.md          ← Zimmer's instructions
    ├── marketing/SKILL.md             ← Tanmay's instructions
    ├── remotion/SKILL.md              ← Leonardo's instructions
    └── ads/SKILL.md                   ← Mark's instructions
```

---

## ⚠️ MANDATORY SKILL READING — NON-NEGOTIABLE

**Every agent MUST read their skill files before doing ANY work. This is not optional.**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kushjain7/redder-bull](https://github.com/kushjain7/redder-bull) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
