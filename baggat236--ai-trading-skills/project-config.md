---
trigger: always_on
description: This skill uses automated data collection:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This repository contains Claude Skills for equity investors and traders. Each skill packages domain-specific prompts, knowledge bases, and helper scripts to assist with market analysis, technical charting, economic calendar monitoring, and trading strategy development. Skills are designed to work in both Claude's web app and Claude Code environments.

⚠️ **Important:** Some skills require paid API subscriptions (FMP API and/or FINVIZ Elite) to function. See the [API Key Management](#api-key-management) section for detailed requirements by skill.

## Repository Architecture

### Skill Structure

Each skill follows a standardized directory structure:

```
<skill-name>/
├── SKILL.md              # Required: Skill definition with YAML frontmatter
├── references/           # Knowledge bases loaded into Claude's context
├── scripts/             # Executable Python scripts (not auto-loaded)
└── assets/              # Templates and resources for output generation
```

**SKILL.md Format:**
- YAML frontmatter with `name` and `description` fields
- `name` must match the directory name for proper skill detection
- Description defines when the skill should be triggered
- Body contains workflow instructions written in imperative/infinitive form
- All instructions assume Claude will execute them, not the user

**Progressive Loading:**
1. Metadata (YAML frontmatter) loads first for skill detection
2. SKILL.md body loads when skill is invoked
3. References load conditionally based on analysis needs
4. Scripts execute on demand, never auto-loaded into context

### Key Design Patterns

**Knowledge Base Organization:**
- `references/` contains markdown files with domain knowledge (sector rotation patterns, technical analysis frameworks, news source credibility guides)
- Knowledge bases provide context without requiring Claude to have specialized training
- References are read selectively during skill execution to minimize token usage

**Script vs. Reference Division:**
- Scripts (`scripts/`) are executable code for API calls, data fetching, report generation
- References (`references/`) are documentation for Claude to read and apply
- Scripts handle I/O; references handle knowledge

**Output Generation:**
- Skills generate reports (markdown + JSON) saved to `reports/` directory
- Filename convention: `<skill>_<analysis-type>_<date>.md` (and `.json`)
- Reports use structured templates from `assets/` directories
- Scripts should default `--output-dir` to `reports/` (or pass `--output-dir reports/` when invoking)

## Common Development Tasks

### Creating a New Skill

Use the skill-creator plugin (available in Claude Code):

```bash
# This invokes the skill-creator to guide you through setup
# Follow the 6-step process: Understanding → Planning → Initializing → Editing → Packaging → Iterating
```

The skill-creator will:
1. Ask clarification questions about the skill's purpose
2. Create the directory structure
3. Generate SKILL.md template
4. Set up references and scripts directories
5. Package the skill into a .skill file

**MANDATORY: After creating or committing a new skill, complete ALL of the following:**

1. **Generate documentation pages** (auto-gen handles EN page + JA stub + index updates):
   ```bash
   python3 scripts/generate_skill_docs.py --skill <skill-name>
   ```
2. **Add to catalog category sections** in `docs/en/skill-catalog.md` and `docs/ja/skill-catalog.md`
3. **Add to API Requirements Matrix** in both catalog files
4. **Add to README** descriptions in `README.md` (English) and `README.ja.md` (Japanese)
5. If the skill requires API keys, add to the API Requirements table in `README.md` and the API要件 section in `README.ja.md`
6. If a new category is needed, create it in both READMEs and both catalogs

> **Pre-commit enforcement:** The `docs-completeness` hook blocks commits if any `skills/*/SKILL.md` exists without corresponding `docs/en/skills/<name>.md` and `docs/ja/skills/<name>.md`. Run the generate command above to fix.

### Creating Documentation Site Pages

Generate documentation pages for the Jekyll site at `docs/`.

**Auto-generation (recommended for most skills):**

```bash
# Generate 6-section EN page + JA stub for a specific skill
# Also updates docs/en/skills/index.md and docs/ja/skills/index.md automatically
python3 scripts/generate_skill_docs.py --skill <skill-name>

# Regenerate all auto-generated pages (ONLY pages marked `generated: true`;
# hand-maintained pages are refused — use --force to override, never in CI)
python3 scripts/generate_skill_docs.py --overwrite
```

> **Skill doc ownership / drift gate:** Committed `docs/{en,ja}/skills/*.md` are
> source-of-truth. A page is generator-owned only if its frontmatter has
> `generated: true`; `generated: false` or an absent marker (and any
> `HAND_WRITTEN` skill) is hand-maintained and **protected** — `--overwrite`
> refuses it (`--force` is the CI-forbidden escape hatch). The
> `skill-docs-drift` pre-commit hook + CI step run `generate_skill_docs.py
> --check`, which content-compares **only** `generated: true` pages and never
> reverts hand-maintained docs. See `docs/README.md` → "Skill Doc Ownership".


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BaggaT236/AI-Trading-Skills](https://github.com/BaggaT236/AI-Trading-Skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
