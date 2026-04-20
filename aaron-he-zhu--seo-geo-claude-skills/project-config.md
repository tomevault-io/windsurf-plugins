---
trigger: always_on
description: Guidelines for AI agents working in this repository.
---

# AGENTS.md

Guidelines for AI agents working in this repository.

## Repository Overview

- **Name**: seo-geo-claude-skills
- **Repository**: https://github.com/aaron-he-zhu/seo-geo-claude-skills
- **Author**: Aaron He Zhu
- **License**: Apache 2.0
- **Skills Specification**: [Agent Skills](https://agentskills.io/specification.md)

This is a content-only repository (no executable code). It contains 20 SEO and GEO skills, 15 one-shot commands, and shared reference documents for AI coding agents.

## Repository Structure

```
seo-geo-claude-skills/
├── research/                         # Phase 1: Market Research (4 skills)
│   ├── keyword-research/SKILL.md
│   ├── competitor-analysis/SKILL.md
│   ├── serp-analysis/SKILL.md
│   └── content-gap-analysis/SKILL.md
├── build/                            # Phase 2: Content Creation (4 skills)
│   ├── seo-content-writer/SKILL.md
│   ├── geo-content-optimizer/SKILL.md
│   ├── meta-tags-optimizer/SKILL.md
│   └── schema-markup-generator/SKILL.md
├── optimize/                         # Phase 3: Improvement (4 skills)
│   ├── on-page-seo-auditor/SKILL.md
│   ├── technical-seo-checker/SKILL.md
│   ├── internal-linking-optimizer/SKILL.md
│   └── content-refresher/SKILL.md
├── monitor/                          # Phase 4: Tracking (4 skills)
│   ├── rank-tracker/SKILL.md
│   ├── backlink-analyzer/SKILL.md
│   ├── performance-reporter/SKILL.md
│   └── alert-manager/SKILL.md
├── cross-cutting/                    # Protocol layer across all phases (4 skills)
│   ├── content-quality-auditor/SKILL.md
│   ├── domain-authority-auditor/SKILL.md
│   ├── entity-optimizer/SKILL.md
│   └── memory-management/SKILL.md
├── commands/                         # 15 one-shot command files
├── references/                       # Shared reference documents
│   ├── core-eeat-benchmark.md        # 80-item content quality framework
│   ├── cite-domain-rating.md         # 40-item domain authority framework
│   ├── skill-contract.md             # Shared skill contract and handoff format
│   └── state-model.md                # Shared memory layout and ownership rules
├── scripts/                          # Developer utilities
│   └── validate-skill.sh             # Validate SKILL.md spec compliance
├── .claude-plugin/                   # Claude Code plugin manifest
│   └── plugin.json
├── marketplace.json                  # skills.sh marketplace manifest
├── .mcp.json                         # Pre-configured MCP servers
├── CLAUDE.md                         # Claude Code auto-loaded agent context
├── AGENTS.md                         # This file (skills.sh convention)
├── CONNECTORS.md                     # Tool placeholder mappings
├── CONTRIBUTING.md                   # Contribution guide
├── VERSIONS.md                       # Version tracking for all skills
├── README.md                         # Main documentation
└── LICENSE                           # Apache 2.0
```

Each skill directory contains a `SKILL.md` and optionally a `references/` subdirectory with skill-specific templates, rubrics, and checklists.

## Skill Categories

| Category | Directory | Purpose | Skills |
|----------|-----------|---------|--------|
| Research | `research/` | Market analysis before content creation | keyword-research, competitor-analysis, serp-analysis, content-gap-analysis |
| Build | `build/` | Content creation optimized for search and AI | seo-content-writer, geo-content-optimizer, meta-tags-optimizer, schema-markup-generator |
| Optimize | `optimize/` | Improve existing content and technical health | on-page-seo-auditor, technical-seo-checker, internal-linking-optimizer, content-refresher |
| Monitor | `monitor/` | Track performance and catch issues | rank-tracker, backlink-analyzer, performance-reporter, alert-manager |
| Cross-cutting | `cross-cutting/` | Protocol layer across all phases | content-quality-auditor, domain-authority-auditor, entity-optimizer, memory-management |

## Hooks & Automation

This library uses prompt-based hooks to automate cross-skill coordination. All hooks are prompt instructions in `hooks/hooks.json` — no shell scripts, no executables.

| Event | Matcher | Behavior |
|-------|---------|----------|
| SessionStart | startup | Load `memory/hot-cache.md` as project context; remind open loops older than 7 days |
| UserPromptSubmit | all | Keep hot-cache priorities in mind silently |
| PostToolUse | Write, Edit | Recommend content-quality-auditor after content writing |
| Stop | all | Prompt to save findings to `memory/`; auto-save veto issues to hot-cache |
| FileChanged | hot-cache.md | Warn if hot-cache exceeds 80 lines or 25KB |

### Hook Design Principles

- **Prompt-based only** — no shell scripts, no executables, no dependencies
- **Transparent** — user can see and override any hook behavior
- **Non-blocking** — hooks suggest actions, they never force them

### Temperature Memory Model

The library uses a three-tier memory model. See [references/state-model.md](https://github.com/aaron-he-zhu/seo-geo-claude-skills/blob/main/references/state-model.md) for full specification.

| Tier | Location | Capacity | Lifecycle |
|------|----------|----------|-----------|
| HOT | `memory/hot-cache.md` | 80 lines | Auto-loaded every session; demotes after 30 days unreferenced |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aaron-he-zhu/seo-geo-claude-skills](https://github.com/aaron-he-zhu/seo-geo-claude-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
