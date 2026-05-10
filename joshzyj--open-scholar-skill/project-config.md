---
trigger: always_on
description: After cloning, run `bash setup.sh` once. This creates symlinks, auto-detects Zotero, and writes `.env`. See `.env.example` for all options.
---

# Open Scholar Skill — Project Instructions

## Getting Started

After cloning, run `bash setup.sh` once. This creates symlinks, auto-detects Zotero, and writes `.env`. See `.env.example` for all options.

---

## Directory Structure

```
open-scholar-skill/
├── CLAUDE.md                    # THIS FILE
├── CHANGELOG.md                 # Version history
├── README.md / USAGE.md         # User-facing docs
├── scripts/gates/               # Executable gate scripts (version-check, safety-scan, verify-citations,
│                                #   pretooluse-data-guard, init-handshake, derive-proj, phase-verify)
├── scripts/init-project.sh      # Project initializer used by scholar-init
├── tests/smoke/                 # Smoke tests (run: bash tests/smoke/run-all.sh)
├── skills/ → .claude/skills/    # Symlink (DO NOT replace with directory)
├── agents/ → .claude/agents/    # Symlink (DO NOT replace with directory)
└── .claude/
    ├── skills/                  # 31 skill directories, each with SKILL.md + references/
    │   ├── _shared/             # Shared protocols (process-logger.md, version-check.md, data-handling-policy.md, tier-b-safety-gate.md)
    │   ├── scholar-init/        # v5.9.0 — project initializer + data safety sidecar populator (4 modes: init/review/add/status)
    │   ├── scholar-analyze/     # Components loaded on-demand via references/component-a-*.md
    │   ├── scholar-auto-improve/# Continuous quality engine (4 modes)
    │   ├── scholar-brainstorm/  # Data-driven RQ generation from codebooks/questionnaires/datasets
    │   ├── scholar-causal/      # 13 strategies loaded on-demand via references/strategies.md
    │   ├── scholar-citation/    # Citation management + verification
    │   ├── scholar-code-review/ # 6-agent code auditor for analysis scripts
    │   ├── scholar-collaborate/ # Multi-author collaboration (CRediT, tasks, mentoring)
    │   ├── scholar-compute/     # 11 modules (on-demand loading via references/module-*.md)
    │   ├── scholar-conceptual/  # Theory building + conceptual diagrams (TikZ/Mermaid)
    │   ├── scholar-data/        # Data collection, open data directory (100+ sources), web scraping
    │   ├── scholar-design/      # Research design + power analysis
    │   ├── scholar-eda/         # Exploratory data analysis
    │   ├── scholar-ethics/      # Research ethics toolkit
    │   ├── scholar-hypothesis/  # Theory + hypothesis formulation
    │   ├── scholar-idea/        # Broad idea → formal RQ (5-agent evaluation panel)
    │   ├── scholar-journal/     # Submission prep (22 journals)
    │   ├── scholar-knowledge/   # User-scoped knowledge graph (INGEST, SEARCH, RELATE, STATUS, EXPORT)
    │   ├── scholar-ling/        # 9 modules loaded on-demand via references/module-*.md
    │   ├── scholar-lit-review/  # Systematic literature review
    │   ├── scholar-lit-review-hypothesis/  # Integrated lit review + hypothesis
    │   ├── scholar-open/        # Open science practices
    │   ├── scholar-openai/      # External review via OpenAI Codex CLI agents
    │   ├── scholar-polish/      # Final prose-level polish (clarity, concision, flow, journal voice)
    │   ├── scholar-qual/        # Qualitative methods (coding, grounded theory, thematic analysis)
    │   ├── scholar-replication/ # Replication package builder + validator
    │   ├── scholar-respond/     # Peer review simulation + R&R
    │   ├── scholar-safety/      # Data safety layer
    │   ├── scholar-verify/      # Two-stage analysis-to-manuscript consistency (4-agent panel)
    │   ├── scholar-write/       # Section drafting (5-agent review panel)
    │   └── sync-docs/           # Cross-document synchronization utility
    └── agents/                  # 19 agents (9 peer-reviewer + 4 verify + 6 code-review)
```

**Version**: v5.12.0 — 32 skills, 19 agents (9 peer-reviewer + 4 verify + 6 code-review)

---

## Key Design Patterns

### Skill Architecture
- Each `SKILL.md` has YAML frontmatter: `name`, `description`, `tools`, `argument-hint`, `user-invocable: true`
- Skills invoked via `/scholar-[name] [arguments]`
- Pattern: dispatch table → numbered workflow steps → save output → quality checklist

### On-Demand Module Loading
Large skills are split into routing stubs + reference files loaded via `cat` on demand:
- **scholar-compute**: 11 module files (`references/module-01-nlp.md` through `module-11-life2vec.md`)
- **scholar-analyze**: 6 component files (`references/component-a-core.md`, etc.) + REVISE-FIGURE mode
- **scholar-causal**: 1 strategies file (`references/strategies.md` — 13 strategies)
- **scholar-ling**: 9 module files (`references/module-01-theory.md` through `module-09-tts-mgt.md`)

Skills route to the correct file, then `cat` only what's needed. This cuts context usage by 60-90%.

### Executable Gate Scripts
Critical gates are enforced by actual scripts in `scripts/gates/`:
- `version-check.sh <dir> <stem>` — prints `SAVE_PATH=...` (prevents overwriting drafts)
- `safety-scan.sh <file>` — local PII/HIPAA detection (exits RED/YELLOW/GREEN); routes to Presidio backend if installed. Binary formats (`.xlsx`, `.parquet`, `.dta`, `.sav`, `.rds`, etc.) are promoted to YELLOW even when scanners return GREEN.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joshzyj/open-scholar-skill](https://github.com/joshzyj/open-scholar-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
