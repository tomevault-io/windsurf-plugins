---
trigger: always_on
description: This skill guides users through a 5-step analysis process using **10+ proven thinking frameworks**:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Deep Reading Analyst** is a Claude AI skill (.skill file) that provides systematic framework for deep analysis of articles, papers, and long-form content using multiple thinking models. It transforms surface-level reading into deep learning through structured workflows.

### Core Purpose

This skill guides users through a 5-step analysis process using **10+ proven thinking frameworks**:

**Quick Analysis (15min)**:
- SCQA Framework (structure thinking)
- 5W2H Analysis (completeness check)

**Standard Analysis (30min)**:
- Critical Thinking (argument evaluation, logical fallacy detection)
- Inversion Thinking (risk and failure mode analysis)

**Deep Analysis (60min)**:
- Mental Models (multi-discipline perspectives: physics, biology, psychology, economics)
- First Principles (assumption stripping, essence extraction)
- Systems Thinking (causal loops, knowledge connections)
- Six Thinking Hats (multi-perspective evaluation)

**Research Level (120min+)**:
- Cross-Source Comparison (multi-article synthesis)

## Repository Structure

### Source Files

- **`src/deep-reading-analyst/`** - Source files for the skill (tracked in Git)
  - `SKILL.md` - Main workflow and decision tree
  - `references/` - Detailed framework documentation (10+ frameworks)
    - `scqa_framework.md` - McKinsey structure thinking (NEW!)
    - `5w2h_analysis.md` - Completeness check with 7 questions (NEW!)
    - `critical_thinking.md` - Argument analysis framework
    - `inversion_thinking.md` - Charlie Munger's risk analysis (NEW!)
    - `mental_models.md` - 30+ models from multiple disciplines (NEW!)
    - `first_principles.md` - Elon Musk's assumption identification
    - `systems_thinking.md` - Relationship mapping and causal loops
    - `six_hats.md` - Edward de Bono's multi-perspective protocol
    - `output_templates.md` - 8 different output formats
    - `comparison_matrix.md` - Multi-source comparison methodology

### Built Package

- **`deep-reading-analyst.skill`** - The packaged skill file (built from `src/`)
  - Format: Standard Claude skill package (ZIP archive)
  - Size: ~40KB (optimized for context window efficiency)
  - Built using: `./build.sh`

### Build Script

- **`build.sh`** - Bash script to package source files into `.skill` file
  - Copies files from `src/deep-reading-analyst/`
  - Creates ZIP archive with proper structure
  - Shows size and contents summary

### Documentation Files

- **`README.md`** - Main documentation with comprehensive usage guide (English)
- **`README_CN.md`** - Complete Chinese documentation with proper spacing
- **`CONTRIBUTING.md`** - Contribution guidelines and skill development rules
- **`CHANGELOG.md`** - Version history
- **`GITHUB_GUIDE.md`** - GitHub repository management guide
- **`examples/EXAMPLES.md`** - Real-world usage examples with timing estimates

### GitHub Configuration

- `.github/ISSUE_TEMPLATE/` - Bug report and feature request templates
- `.github/PULL_REQUEST_TEMPLATE.md` - PR template

## Development Workflow

### Editing the Skill

**Standard workflow:**

```bash
# 1. Edit source files directly
vim src/deep-reading-analyst/SKILL.md
vim src/deep-reading-analyst/references/critical_thinking.md

# 2. Rebuild the skill package
./build.sh

# 3. Test by importing the .skill file into Claude

# 4. Commit both source and built files
git add src/ deep-reading-analyst.skill
git commit -m "feat: add new thinking framework"
```

**Examining the packaged skill:**

```bash
# List contents of the skill archive
unzip -l deep-reading-analyst.skill

# View specific file without extracting
unzip -p deep-reading-analyst.skill deep-reading-analyst/SKILL.md
```

### Key Architectural Decisions

1. **Layered Framework Loading**: The skill uses progressive framework loading based on analysis depth:
   - Level 1 (Quick): Structure only - 15 min
   - Level 2 (Standard): + Critical Thinking - 30 min
   - Level 3 (Deep): + First Principles + Systems + Six Hats - 60 min
   - Level 4 (Research): + Cross-source comparison via web_search - 120+ min

2. **Reference Documents as Knowledge Base**: Instead of embedding all frameworks in SKILL.md, they're separated into `references/` for:
   - Context window efficiency
   - Modular updates
   - Clearer framework documentation

3. **Workflow-Driven Design**: The SKILL.md follows a strict 5-step process:
   - Step 1: Initialize (gather user goals and depth preference)
   - Step 2: Structural Understanding (always performed)
   - Step 3: Apply Thinking Models (based on depth level)
   - Step 4: Synthesis & Output (customized to user goal)
   - Step 5: Knowledge Activation (actionable next steps)

## Skill Trigger Conditions

The skill auto-triggers when users:

- Say: "analyze this article", "help me understand", "deep dive into", "extract insights from"
- Provide URLs or long-form content for analysis
- Request specific frameworks: "use critical thinking", "apply first principles"

## Adding New Features

### Adding a New Thinking Framework

1. Create `src/deep-reading-analyst/references/new_framework.md` following this structure:
   - Overview: What is this framework?

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ginobefun/deep-reading-analyst-skill](https://github.com/ginobefun/deep-reading-analyst-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
