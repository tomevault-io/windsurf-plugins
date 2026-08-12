---
trigger: always_on
description: Guidelines for AI agents working in this repository.
---

# AGENTS.md - Instructions for AI Agents

Guidelines for AI agents working in this repository.

## Repository Overview

This repository contains **Agent Skills** for AI agents following the [Agent Skills specification](https://agentskills.io/specification.md). Skills install to `.agents/skills/` (the cross-agent standard). This repo also serves as a **Claude Code plugin marketplace** via `.claude-plugin/marketplace.json`.

- **Name**: Finance Skills
- **GitHub**: [GAJETOso/financeskills](https://github.com/GAJETOso/financeskills)
- **Creator**: KOMVIA
- **License**: MIT

## Repository Structure

```
financeskills/
├── .claude-plugin/
│   ├── plugin.json        # Claude Code plugin manifest
│   └── marketplace.json   # Claude Code plugin marketplace manifest
├── skills/                # Agent Skills
│   └── skill-name/
│       ├── SKILL.md       # Required skill file
│       └── evals/         # Required evaluation suite
├── tools/
│   ├── finance_helpers.py # Shared python utilities
│   └── REGISTRY.md        # Tool index with capabilities
├── foundations/           # Core technical guides
├── compliance/            # Regulatory mapping (IFRS/GAAP)
├── CONTRIBUTING.md
├── LICENSE
└── README.md
```

## Build / Lint / Test Commands

**Skills** are content-only (no build step). Verify manually:
- YAML frontmatter is valid
- `name` field matches directory name exactly
- `name` is 1-64 chars, lowercase alphanumeric and hyphens only
- `description` is 1-1024 characters

**Validation** - run before committing:
- `python3 validate_skills.py` - enforces this spec: frontmatter constraints, name-dir match, evals.json schema, fixture existence, dead-link detection, <500-line limit, script self-tests, marketplace sync, no compiled artifacts. (`./validate-skills.sh` wraps it.)

**Evals** (`skills/*/evals/evals.json`) are runnable benchmarks:
- `python3 tools/evals/run_evals.py --dry-run` - schema + fixture check (no model calls)
- `python3 tools/evals/run_evals.py --skill <name>` - run one skill's evals against a model (needs `ANTHROPIC_API_KEY`, or `--backend cli` for the Claude Code CLI)
- `python3 tools/evals/run_evals.py --all` - full regression run; reports land in `tools/evals/results/`
- Eval `files` entries are paths relative to the skill directory (fixtures live in `evals/files/`)

**Scripts** (`skills/*/scripts/calculate.py`): 38 of 43 skills ship deterministic calculators with self-tests. Always import these for computations instead of doing mental math - per CLAUDE.md, calculations must be verified with code. The 5 skills without scripts (audit-checklist, sox-compliance, close-management, esg-reporting, nlp-earnings-sentiment) are judgment-based by design.

**Standards summaries** (`standards/`) carry an `as_of` date and a verification footer. Treat them as orientation, not authority - verify against the issuing body's current text for decision-grade output.

## Agent Skills Specification

Skills follow the [Agent Skills spec](https://agentskills.io/specification.md).

### Required Frontmatter

```yaml
---
name: skill-name
description: What this skill does and when to use it. Include trigger phrases.
---
```

### Frontmatter Field Constraints

| Field         | Required | Constraints                                                      |
|---------------|----------|------------------------------------------------------------------|
| `name`        | Yes      | 1-64 chars, lowercase `a-z`, numbers, hyphens. Must match dir.   |
| `description` | Yes      | 1-1024 chars. Describe what it does and when to use it.          |
| `license`     | No       | License name (default: MIT)                                      |
| `metadata`    | No       | Key-value pairs (author, version, etc.)                          |

### Name Field Rules

- Lowercase letters, numbers, and hyphens only
- Cannot start or end with hyphen
- No consecutive hyphens (`--`)
- Must match parent directory name exactly

**Valid**: `tax-planning`, `financial-analysis`, `risk-assessment`
**Invalid**: `Tax-Planning`, `-tax`, `tax--planning`

### Optional Skill Directories

```
skills/skill-name/
├── SKILL.md        # Required - main instructions (<500 lines)
├── evals/          # Required - evaluation suite
│   ├── evals.json  # Prompts, expected outputs, assertions
│   └── files/      # Fixture data (CSVs etc.) referenced by evals
├── references/     # Optional - detailed docs loaded on demand
├── scripts/        # Optional - executable code (calculate.py with self-tests)
└── assets/         # Optional - output templates, data files
```

## Writing Style Guidelines

### Structure

- Keep `SKILL.md` under 500 lines (move details to `references/`)
- Use H2 (`##`) for main sections, H3 (`###`) for subsections
- Use bullet points and numbered lists liberally
- Short paragraphs (2-4 sentences max)

### Tone

- Direct and instructional
- Second person ("You are a senior financial analyst")
- Professional but approachable

### Formatting

- Bold (`**text**`) for key terms
- Code blocks for examples and templates
- Tables for reference data
- No excessive emojis

### Clarity Principles

- Clarity over cleverness
- Specific over vague
- Active voice over passive
- One idea per section

### Description Field Best Practices


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GAJETOso/financeskills](https://github.com/GAJETOso/financeskills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
