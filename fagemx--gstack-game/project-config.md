---
trigger: always_on
description: bun run build                    # generate all SKILL.md from templates
---

# gstack-game development

## Commands

```bash
bun run build                    # generate all SKILL.md from templates
bun run gen:skill-docs           # same as build
bun run gen:skill-docs:check     # check for drift without writing (CI use)
bun test                         # run Tier 1 validation tests (free, <2s)
```

## Testing

```bash
bun test                         # run before every commit — free, <2s
```

`bun test` runs template validation: frontmatter checks, preamble injection verification,
placeholder expansion, drift detection, and tier validation. All 14 tests must pass before committing.

## Project structure

```
gstack-game/
├── CLAUDE.md                    ← this file (dev handoff)
├── README.md                    ← user-facing docs (EN)
├── README.zh-TW.md             ← user-facing docs (繁中)
├── ETHOS.md                     ← game dev philosophy
├── CHANGELOG.md                 ← version history (user-facing)
├── VERSION                      ← current version (0.3.0)
├── package.json                 ← build scripts
├── bin/                         ← shared utilities
│   ├── install.sh               ← umbrella installer
│   ├── gstack-config            ← config read/write
│   ├── gstack-diff-scope        ← game-aware diff classification (11 scopes)
│   ├── gstack-review-log        ← review logging
│   ├── gstack-review-read       ← review dashboard
│   ├── gstack-telemetry-log     ← telemetry
│   └── gstack-slug              ← repo slug detection
├── scripts/
│   └── gen-skill-docs.ts        ← template engine (SKILL.md.tmpl → SKILL.md)
├── skills/                      ← 28 published skills + shared/
│   ├── shared/
│   │   ├── preamble-core.md     ← T1+: bash setup, artifacts, completion status
│   │   ├── preamble-standard.md ← T2+: voice, AskUser, vocabulary, routing
│   │   ├── preamble-expert.md   ← T3: scope drift, review staleness
│   │   └── preamble-telemetry.md← all: end-of-session telemetry (always last)
│   ├── game-review/             ← GDD review (255L, 80%)
│   ├── balance-review/          ← economy & balance (286L, 70%)
│   ├── player-experience/       ← player walkthrough (273L, 75%)
│   ├── pitch-review/            ← pitch evaluation (302L, 70%)
│   ├── gameplay-implementation-review/        ← PR review (186L, 75%)
│   ├── game-ideation/           ← concept brainstorming (524L, 65%)
│   ├── game-direction/          ← direction review (490L, 55%)
│   ├── game-eng-review/         ← tech architecture (462L + 5 refs, 70%)
│   ├── game-qa/                 ← QA testing (702L, 65%)
│   ├── game-ux-review/          ← UI/UX (565L, 60%)
│   ├── plan-design-review/      ← pre-impl design plan review (679L + 5 refs, 65%)
│   ├── game-ship/               ← release process (448L, 65%)
│   ├── game-import/             ← project import (514L)
│   ├── triage/                  ← project navigator (320L)
│   ├── feel-pass/               ← game feel diagnosis (280L)
│   ├── build-playability-review/← playability assessment (211L)
│   ├── prototype-slice-plan/    ← prototype planning (235L)
│   ├── implementation-handoff/  ← implementation handoff (225L)
│   ├── game-debug/              ← debugging (182L, 55%)
│   ├── game-retro/              ← retrospective (166L, 40%)
│   ├── game-codex/              ← adversarial review (331L + 4 refs, 70%)
│   ├── game-docs/               ← release docs (137L, 40%)
│   ├── game-visual-qa/          ← visual QA (231L + 5 refs, 60%)
│   ├── asset-review/            ← asset pipeline (329L + 5 refs, 70%)
│   ├── playtest/                ← playtest protocol (251L + 3 refs, 65%)
│   ├── careful/                 ← destructive cmd safety (62L)
│   ├── guard/                   ← full safety mode (56L)
│   └── unfreeze/                ← unlock guard (32L)
├── .claude/skills/              ← 6 internal maintenance skills (not published)
│   ├── skill-review/            ← skill quality review (431L)
│   ├── contribute-review/       ← contribution review (334L)
│   ├── issue-create/            ← create GitHub issues from conversation
│   ├── issue-plan/              ← three-phase deep-dive planning (research → innovate → plan)
│   ├── issue-action/            ← implement from approved plan → PR
│   └── pr-review-loop/          ← automated PR review-fix cycle (max 3 iterations)
├── test/
│   └── gen-skill-docs.test.ts   ← Tier 1 template validation (11 tests)
└── docs/
    ├── DEVELOPMENT.md           ← full project overview, skill map, migration guide
    ├── domain-judgment-gaps.md  ← expert calibration checklist
    └── source-quality-assessment.md ← quality comparison of 3 sources
```

## SKILL.md workflow

SKILL.md files are **generated** from `.tmpl` templates. Never edit SKILL.md directly.

1. Edit the `.tmpl` file (e.g. `skills/game-review/SKILL.md.tmpl`)
2. Run `bun run build`
3. Commit both the `.tmpl` and generated `.md` files

**Merge conflicts on SKILL.md files:** Never resolve conflicts on generated SKILL.md
files by accepting either side. Instead: (1) resolve conflicts on the `.tmpl` templates,
(2) run `bun run build` to regenerate, (3) stage the regenerated files.

## Writing SKILL templates

SKILL.md.tmpl files are **prompt templates read by Claude**, not bash scripts.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fagemx/gstack-game](https://github.com/fagemx/gstack-game) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
