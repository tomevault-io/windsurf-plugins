---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
Most of these skills have also been tested in OpenAI Codex, Google Gemini, Cursor, Perplexity, and Hermes and have been shown to work if the AGENTS.md file is copied to the AGENTS.md file.
Note that the skills that use image understanding to understand MicroSim layout errors
do not work well on non-Codex systems.

## Project Overview

This repository contains a curated collection of Codex AI skills for creating intelligent, interactive educational textbooks. Each skill is an autonomous agent that automates specific aspects of educational content creation - from learning graph generation to interactive p5.js simulations (MicroSims).

The skills are designed for building **Level 2+ intelligent textbooks**.  Level 2 puts a focus on generating interactive content that can be monitored using the experience API (xAPI).  Books are created using markdown, MkDocs with Material theme.
Content generated should following educational frameworks like Bloom's Taxonomy (2001) for classifying educationa objecives.
The levels include remember, understand, apply, analyze, evaluate and create.  Defintions of terms should follow the ISO 11179 metadata standards (precise, concise, distinct, non-circular, unecumbered with rules).  All books
contain a concept dependency graphs in the docs/learing-graph/learning-graph.json file.  This helps us
not use concepts in a chapter that have not been introduced yet.

## Repository Structure

```
Codex-skills/
├── skills/                          # Active skill definitions (14 loaded skills)
│   ├── archived/                    # Verbatim originals of consolidated skills (never loaded; see its README alias map)
│   │
│   │ # Meta-Skills (routers with references/ guides)
│   ├── book-installer/              # Infrastructure: init-textbook scaffold (feature 0), 40 features incl. Google Analytics, book-metrics
│   ├── microsim-generator/          # MicroSims: p5, chartjs, timeline, map, vis-network, mermaid, causal-loop, concept-classifier, infographic-overlay, docker-python-lab, verified-infographic posters, …
│   ├── microsim-utils/              # MicroSim QA: standardization, screen-capture, index-generator, iframe tools, layout-reviewer, diagram-reports
│   ├── book-media-generator/        # Media: MARP web decks, .pptx lectures, stories, chapter images, TTS + pronounce buttons
│   ├── book-publisher/              # Promotion: README, LinkedIn post, LinkedIn carousel, press release (all read book-metrics.json)
│   │
│   │ # Content-Pipeline Skills (kept separate — complex workflows)
│   ├── course-description-analyzer/ # Validates course descriptions
│   ├── learning-graph-generator/   # Generates 200-concept learning graphs (+ analyze/convert/taxonomy scripts)
│   ├── book-chapter-generator/      # Designs chapter structure from learning graph
│   ├── chapter-content-generator/   # Generates detailed chapter content (canonical blooms-taxonomy.md lives here)
│   ├── glossary-generator/         # Creates ISO 11179-compliant glossaries
│   ├── faq-generator/              # Generates FAQs from course content
│   ├── quiz-generator/             # Generates Bloom's Taxonomy-aligned quizzes
│   ├── reference-generator/        # Generates curated reference lists
│   │
│   │ # Standalone
│   └── docx-to-web-publisher/      # .docx → Next.js content-catalog pages (non-MkDocs)
│
├── docs/                          # MkDocs documentation site
├── scripts/                       # Utility scripts
│   └── bk-install-skills         # Symlinks skills into every agent present
├── commands/                      # Slash commands
│   └── ibook.md                  # /ibook runbook command
└── mkdocs.yml                    # MkDocs configuration
```

**Key Directories:**
- **`skills/`**: Each subdirectory contains a SKILL.md file defining the skill's behavior, plus supporting files (Python scripts, templates, reference docs)
- **`skills/archived/`**: Contains individual skills that have been consolidated into meta-skills to stay under the 30-skill limit
- **`docs/`**: Documentation site built with MkDocs Material theme, deployed to GitHub Pages

## Architecture Patterns

### Skill System

Skills are autonomous agents loaded by Codex. Each skill:
1. Is defined by a `SKILL.md` file with YAML frontmatter containing `name:`, `description:`, `license:`, and optional `allowed-tools:`
2. Contains workflow instructions that Codex executes step-by-step
3. May include supporting assets (Python scripts, templates, reference documents)
4. Is designed to be invoked with `/skill [skill-name]` or through the Skill tool

### Meta-Skill Architecture (30-Skill Limit Workaround)

Codex has a **maximum limit of 30 skills** that can be loaded at once. To work around this limitation, related skills have been consolidated into **meta-skills** that act as routers:

| Meta-Skill | Sub-Skills (in `references/` folder) | Purpose |
|------------|--------------------------------------|---------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dmccreary/ibook-skills](https://github.com/dmccreary/ibook-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
