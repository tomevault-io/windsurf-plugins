---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

<!-- AUTO:START:overview -->
A documentation-only repository containing 27 guides, a skills library of 352 agentic skills, 72 agent definitions, and 17 team compositions following the [Agent Skills open standard](https://agentskills.io). There is no build system, no tests, and no compiled code — all content is markdown and YAML.

The guides serve as the human entry point to the agentic system: practical workflows explaining when, why, and how to interact with agents, teams, and skills through Claude Code.
<!-- AUTO:END:overview -->

## Architecture

### Four Content Types

1. **Guides** (`guides/` directory): Human-readable documentation organized into five categories (workflow, infrastructure, reference, design, investigation). Each guide has YAML frontmatter (`title`, `description`, `category`, `agents`, `teams`, `skills`) and follows a standard template (`guides/_template.md`). Guides serve as the human entry point to the agentic system.

2. **Skills** (`skills/` directory): Machine-consumable structured procedures that agentic systems execute. Each skill lives at `skills/<skill-name>/SKILL.md` with YAML frontmatter (`name`, `description`, `allowed-tools`, `metadata`) and standardized sections (When to Use, Inputs, Procedure, Validation, Common Pitfalls, Related Skills). Skills are organized into 62 logical domains via metadata tags, but the directory structure is flat.

3. **Agents** (`agents/` directory): Persona definitions for Claude Code subagents. Each agent is a markdown file with YAML frontmatter (`name`, `description`, `tools`, `model`, `priority`) defining *who* handles a task. Agents span development, compliance, review, project management, DevOps, MLOps, workflow visualization, esoteric, and specialty domains.

4. **Teams** (`teams/` directory): Predefined multi-agent compositions for complex workflows. Each team is a markdown file with YAML frontmatter (`name`, `description`, `lead`, `members[]`, `coordination`) and an embedded machine-readable configuration block. Teams define *who works together* — coordinated groups of agents with assigned roles and a defined coordination pattern.

These four types complement each other: skills define *how* (procedure, validation, recovery), agents define *who* (persona, tools, style), teams define *who works together* (composition, roles, coordination), and guides provide the background knowledge all draw from.

### Registries

<!-- AUTO:START:registries -->
- `skills/_registry.yml` is the machine-readable catalog of all 352 skills across 64 domains: r-packages (10), jigsawr (5), containerization (10), reporting (4), compliance (17), mcp-integration (6), web-dev (4), git (7), general (23), citations (3), data-serialization (2), review (11), bushcraft (4), esoteric (29), design (5), defensive (6), project-management (6), devops (13), observability (13), edge-computing (1), mlops (12), workflow-visualization (6), swarm (9), morphic (7), alchemy (4), tcg (3), intellectual-property (4), web-scraping (2), gardening (5), shiny (7), animal-training (2), mycology (2), prospecting (2), crafting (1), library-science (3), linguistics (1), travel (6), relocation (3), a2a-protocol (3), geometry (3), number-theory (3), stochastic-processes (3), theoretical-science (3), diffusion (4), hildegard (5), maintenance (5), blender (3), visualization (4), 3d-printing (3), lapidary (4), entomology (5), versioning (4), spectroscopy (6), chromatography (5), gpu-optimization (2), digital-logic (4), electromagnetism (4), levitation (3), i18n (1), synoptic (4), tensegrity (1), cli (4), open-source (2), investigation (6).
- `agents/_registry.yml` is the machine-readable catalog of all 72 agents.
- `teams/_registry.yml` is the machine-readable catalog of all 17 teams.
- `guides/_registry.yml` is the machine-readable catalog of all 27 guides across 5 categories.

When adding or removing skills, agents, teams, or guides, the corresponding registry must be updated to stay in sync.
<!-- AUTO:END:registries -->

### Plugin Packaging

The repository is packaged as a Claude Code plugin via `.claude-plugin/plugin.json`. When installed, Claude Code auto-discovers skills (`skills/*/SKILL.md`) and agents (`agents/*.md`). Teams are bundled but not auto-discovered — they require activation via `TeamCreate` and the CLAUDE.md activation instruction. The plugin can be installed via a local marketplace (see README.md for setup). Validation: `claude plugin validate /path/to/agent-almanac`.

### Cross-References

Guides, skills, agents, and teams are cross-referenced. The parent project `CLAUDE.md` at `/mnt/d/dev/p/CLAUDE.md` references several guides via `@agent-almanac/guides/` paths. Skills reference related skills by relative path. Teams reference their member agents. The project `.claude/agents/` symlinks to `agents/` for Claude Code discovery.

## Editing Conventions

- SKILL.md files must retain the YAML frontmatter delimited by `---` and all standardized sections
- Each Procedure step uses the pattern: numbered step with sub-steps, then `**Expected:**` and `**On failure:**` blocks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pjt222/agent-almanac](https://github.com/pjt222/agent-almanac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
