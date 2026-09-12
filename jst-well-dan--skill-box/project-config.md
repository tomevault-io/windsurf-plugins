---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the Claude Skills Vault, a curated collection of 47+ practical Claude Skills organized across 8 categories. The repository serves as both a marketplace of skills and a framework for managing and distributing them. Skills are modular packages that extend Claude's capabilities with specialized knowledge, workflows, and tool integrations.

## Repository Structure

```
awesome-claude-skills/
├── .claude-plugin/
│   └── marketplace.json          # Central registry of all 47+ skills
│
├── Category Directories/          # Skills organized by category
│   ├── ai-meta/                   # Skill creation and MCP building
│   ├── business-analyst/          # Data and finance tools
│   ├── content-pipeline/          # Scraping and publishing
│   ├── dev-tools/                 # Engineering and CLI tools
│   ├── obsidian/                  # Obsidian-specific integrations
│   ├── productivity/              # Office and efficiency tools
│   └── visual-creative/           # Art and design tools
│
├── scripts/                       # Python automation for skill management
│   ├── fetch_external_skills.py  # Main orchestrator for fetching external skills
│   ├── github_fetcher.py         # GitHub API interactions
│   ├── marketplace_updater.py    # Updates marketplace.json
│   ├── skill_processor.py        # Validates and processes skills
│   └── utils.py                  # Common utilities
│
├── config/
│   └── external_skills_config.json  # Configuration for external skill sources
│
├── awesome-skills-showcase/      # React/Vite web showcase application
│   ├── src/                      # TypeScript/React source code
│   └── package.json              # Node.js dependencies and scripts
│
└── requirements.txt              # Python dependencies
```

## Skill Structure

Every skill follows a standardized structure:

```
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter with name and description
│   └── Markdown instructions
└── Optional directories:
    ├── scripts/      # Executable code (Python/Bash)
    ├── references/   # Documentation loaded on demand
    └── assets/       # Templates, images, etc.
```

The `name` and `description` in YAML frontmatter are critical - they determine when Claude activates the skill.

## Python Environment and Scripts

### Dependencies

Install Python dependencies before running scripts:
```bash
pip install -r requirements.txt
```

Required packages:
- `requests>=2.31.0` - HTTP requests for GitHub API
- `python-frontmatter>=1.1.0` - Parse YAML frontmatter in SKILL.md files
- `PyYAML>=6.0.1` - YAML parsing
- `jsonschema>=4.20.0` - Validate JSON schemas

### Fetching External Skills

The primary automation tool is `fetch_external_skills.py`, which fetches skills from external GitHub repositories and integrates them into the marketplace.

**Basic usage:**
```bash
# Fetch all configured skills
python scripts/fetch_external_skills.py --all

# Fetch specific skill by ID
python scripts/fetch_external_skills.py --skill markdown-to-epub-converter

# Dry run (simulate without writing)
python scripts/fetch_external_skills.py --dry-run --all

# Force re-fetch existing skills
python scripts/fetch_external_skills.py --force --all

# Use custom config file
python scripts/fetch_external_skills.py --config path/to/config.json
```

**Configuration:**
External skills are defined in `config/external_skills_config.json`. Each skill entry includes:
- `id` - Unique identifier
- `github_url` - Source repository URL
- `repo_type` - Type: `standalone`, `multi_skill`
- `target_folder` - Destination directory
- `category` - Marketplace category
- `extraction_config` - How to extract from repo:
  - `full_repo` - Clone entire repository
  - `subfolder` - Extract specific subfolder
  - `deep_nested` - Extract deeply nested subfolder

### Script Architecture

The fetching system is modular:

1. **fetch_external_skills.py** - Main orchestrator
   - Loads configuration
   - Validates environment
   - Coordinates fetching workflow
   - Generates execution reports

2. **github_fetcher.py** - GitHub interactions
   - Handles GitHub API authentication (uses `GITHUB_TOKEN` env var)
   - Downloads repositories via API or git clone
   - Extracts specific subfolders based on config

3. **skill_processor.py** - Skill validation
   - Validates SKILL.md structure and frontmatter
   - Checks for required metadata (name, description)
   - Normalizes skill metadata

4. **marketplace_updater.py** - Registry management
   - Updates `.claude-plugin/marketplace.json`
   - Merges new skills with existing ones
   - Maintains marketplace schema

## Web Showcase Application

The `awesome-skills-showcase/` directory contains a React/Vite web application for browsing skills.

### Development Commands

```bash
# Navigate to showcase directory
cd awesome-skills-showcase

# Install dependencies (if not already installed)
pnpm install

# Start development server (default: http://localhost:5173)
pnpm dev

# Build for production
pnpm build

# Type check TypeScript
pnpm build  # Includes tsc -b

# Lint code
pnpm lint

# Preview production build

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jst-Well-Dan/Skill-Box](https://github.com/Jst-Well-Dan/Skill-Box) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
