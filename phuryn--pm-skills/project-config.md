---
trigger: always_on
description: Guidance for AI agents (Claude Code, Cowork, and others) working in this repository. This file is the single source of truth for how the project is structured and maintained.
---

# CLAUDE.md

Guidance for AI agents (Claude Code, Cowork, and others) working in this repository. This file is the single source of truth for how the project is structured and maintained.

## Project Overview

**PM Skills** (`phuryn/pm-skills`) — a marketplace of **9 independent plugins** (68 skills, 42 commands) that bring structured product-management workflows to AI coding assistants. Built for Claude Code and Claude Cowork; the skills are also compatible with other agents (Gemini CLI, Cursor, Codex CLI).

Owner: Paweł Huryn — pawel@productcompass.pm — https://www.productcompass.pm

## Repo Structure

```
pm-skills/                           <- repo root
├── .claude-plugin/marketplace.json  <- root marketplace manifest (lists all 9 plugins)
├── .docs/images/                    <- images used by README (webp, gif)
├── .gitattributes
├── .gitignore
├── .github/workflows/               <- CI: tests.yml (every PR/push), tag-on-merge.yml (auto-release)
├── CHANGELOG.md                     <- release source of truth (new ## vX.Y.Z heading on main = release)
├── CLAUDE.md                        <- this file (agent guidance, single source of truth)
├── AGENTS.md                        <- pointer to CLAUDE.md (for non-Claude agents)
├── CONTRIBUTING.md                  <- contributor guidelines
├── README.md                        <- public documentation (GitHub)
├── LICENSE                          <- MIT
├── validate_plugins.py              <- plugin validator
├── tests/                           <- unit + docs-consistency tests (unittest)
└── pm-{name}/                       <- 9 plugin directories
    ├── .claude-plugin/plugin.json   <- per-plugin manifest
    ├── skills/{skill}/SKILL.md      <- one folder per skill
    ├── commands/{command}.md        <- one file per command
    └── README.md                    <- per-plugin documentation
```

### The 9 plugins

| Plugin | Focus |
|--------|-------|
| `pm-product-discovery` | Ideation, experiments, assumption testing, prioritization, interview synthesis |
| `pm-product-strategy` | Vision, strategy/lean/business-model canvas, SWOT, PESTLE, Ansoff, Porter, monetization |
| `pm-execution` | PRDs, OKRs, roadmaps, sprints, pre-mortems, stakeholder maps, user stories, red-teaming |
| `pm-market-research` | Personas, segmentation, sentiment analysis, competitive analysis, market sizing |
| `pm-data-analytics` | SQL query generation, cohort/retention analysis |
| `pm-go-to-market` | GTM strategy, growth loops, motions, beachhead segments, ICPs |
| `pm-marketing-growth` | Marketing ideas, value-prop statements, North Star metrics, naming, positioning |
| `pm-toolkit` | Resume review, NDA drafting, privacy policy, grammar/flow checking |
| `pm-ai-shipping` | AI Shipping Kit: document a vibe-coded app, map test coverage, audit security/performance against intended behavior, compile a shipping packet |

## Key Design Rules

- **Skills = nouns/concepts.** Frameworks and analytical knowledge Claude auto-loads when the topic matches (`lean-canvas`, `pre-mortem`, `market-sizing`).
- **Commands = verbs.** User-triggered workflows that chain one or more skills (`/write-prd`, `/discover`, `/plan-launch`).
- **No cross-plugin references.** Commands suggest follow-ups in natural language only ("Want me to design growth loops?"). Never hard-reference a command from another plugin — plugins install independently, so a hard reference can break.
- **Intra-plugin "Uses" references are fine** — skills and commands in the same plugin always ship together.
- Commands use a single `$ARGUMENTS` placeholder. Skills need no placeholders (they read context from the conversation).
- **Frontmatter required:** Skills need `name` + `description`; commands need `description` + `argument-hint`.
- A skill's `name` **must match its directory name**.
- Skills can be force-loaded with `/plugin-name:skill-name` or `/skill-name`.
- Keep frontmatter lean (always loaded); put detail in the SKILL.md body (loaded when triggered) — progressive disclosure.

## What's Visible Where

| Location | Visible in | Notes |
|----------|-----------|-------|
| `marketplace.json` → `description` | Cowork marketplace browser, Claude Code | One-liner for the whole marketplace |
| `plugin.json` → `description` | Cowork plugin list, Claude Code | Per-plugin summary; concise and functional |
| `SKILL.md` frontmatter → `description` | Cowork skill list, Claude auto-loading | Include trigger phrases so Claude loads the skill at the right time |
| Command frontmatter → `description` + `argument-hint` | Cowork and Claude Code (typing `/`) | Short and actionable |
| `README.md` (repo root) | GitHub only | Full docs; not loaded by Claude at runtime |

Descriptions in `plugin.json` and the repo `README.md` should stay aligned (identical text).

## Versioning & Releases

- **`CHANGELOG.md` is the source of truth.** The newest `## vX.Y.Z — YYYY-MM-DD` heading is the released version. Pushing a commit to `main` that adds a new heading makes CI (`.github/workflows/tag-on-merge.yml`) verify the version sync and test suite, tag `vX.Y.Z`, and publish a GitHub Release with that section as notes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phuryn/pm-skills](https://github.com/phuryn/pm-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
