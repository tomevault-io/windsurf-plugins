---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains a curated collection of Claude AI skills for creating intelligent, interactive educational textbooks. Each skill is an autonomous agent that automates specific aspects of educational content creation - from learning graph generation to interactive p5.js simulations (MicroSims).

The skills are designed for building **Level 2+ intelligent textbooks**.  Level 2 puts a focus on generating interactive content that can be monitored using the experience API (xAPI).  Books are created using markdown, MkDocs with Material theme.
Content generated should following educational frameworks like Bloom's Taxonomy (2001) for classifying educationa objecives.
The levels include remember, understand, apply, analyze, evaluate and create.  Defintions of terms should follow the ISO 11179 metadata standards (precise, concise, distinct, non-circular, unecumbered with rules).  All books
contain a concept dependency graphs in the docs/learing-graph/learning-graph.json file.  This helps us
not use concepts in a chapter that have not been introduced yet.

## Repository Structure

```
claude-skills/
├── skills/                          # Active skill definitions
│   ├── archived/                    # Archived skills (consolidated into meta-skills)
│   │
│   │ # Meta-Skills (consolidate multiple sub-skills)
│   ├── book-installer/              # Routes to: mkdocs-template, learning-graph-viewer, skill-tracker
│   ├── microsim-generator/          # Routes to: p5, chartjs, timeline, map, vis-network, mermaid, etc.
│   ├── microsim-utils/              # Routes to: standardization, screen-capture, add-icons, index-generator
│   │
│   │ # Book Generation Skills
│   ├── book-chapter-generator/      # Designs chapter structure from learning graph
│   ├── book-metrics-generator/      # Analyzes book content and generates metrics
│   ├── chapter-content-generator/   # Generates detailed chapter content
│   ├── course-description-analyzer/ # Validates course descriptions
│   ├── diagram-reports-generator/   # Creates diagram/MicroSim reports
│   ├── faq-generator/              # Generates FAQs from course content
│   ├── glossary-generator/         # Creates ISO 11179-compliant glossaries
│   ├── learning-graph-generator/   # Generates 200-concept learning graphs
│   │   ├── SKILL.md               # Skill definition and workflow
│   │   ├── analyze-graph.py       # Validates DAG structure and quality
│   │   ├── csv-to-json.py         # Converts CSV to vis-network JSON
│   │   ├── add-taxonomy.py        # Adds taxonomy column to CSV
│   │   └── taxonomy-distribution.py # Generates taxonomy reports
│   ├── linkedin-announcement-generator/ # Creates LinkedIn posts for books
│   ├── quiz-generator/             # Generates Bloom's Taxonomy-aligned quizzes
│   ├── readme-generator/           # Creates GitHub README files
│   ├── reference-generator/        # Generates curated reference lists
│   │
│   │ # Specialized Skills
│   ├── concept-classifier/         # Creates classification quiz MicroSims
│   ├── moving-rainbow/             # MicroPython for Raspberry Pi Pico
│   └── vis-network/                # vis-network specific utilities
│
├── docs/                          # MkDocs documentation site
├── scripts/                       # Utility scripts
│   ├── install-claude-skills.sh  # Creates symlinks to ~/.claude/skills/
│   ├── list-skills.sh            # Lists available skills
│   └── list-skills-format.sh     # Outputs skills in various formats
├── commands/                      # Slash commands
│   └── skills.md                 # /skills command definition
└── mkdocs.yml                    # MkDocs configuration
```

**Key Directories:**
- **`skills/`**: Each subdirectory contains a SKILL.md file defining the skill's behavior, plus supporting files (Python scripts, templates, reference docs)
- **`skills/archived/`**: Contains individual skills that have been consolidated into meta-skills to stay under the 30-skill limit
- **`docs/`**: Documentation site built with MkDocs Material theme, deployed to GitHub Pages

## Architecture Patterns

### Skill System

Skills are autonomous agents loaded by Claude Code. Each skill:
1. Is defined by a `SKILL.md` file with YAML frontmatter containing `name:`, `description:`, `license:`, and optional `allowed-tools:`
2. Contains workflow instructions that Claude executes step-by-step
3. May include supporting assets (Python scripts, templates, reference documents)
4. Is designed to be invoked with `/skill [skill-name]` or through the Skill tool

### Meta-Skill Architecture (30-Skill Limit Workaround)

Claude Code has a **maximum limit of 30 skills** that can be loaded at once. To work around this limitation, related skills have been consolidated into **meta-skills** that act as routers:

| Meta-Skill | Sub-Skills (in `references/` folder) | Purpose |
|------------|--------------------------------------|---------|
| `book-installer` | mkdocs-template, learning-graph-viewer, skill-tracker | Project setup and infrastructure |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dmccreary/claude-skills](https://github.com/dmccreary/claude-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
