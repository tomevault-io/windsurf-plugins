---
trigger: always_on
description: This file provides guidance to Agents when working with this repository.
---

# AGENTS.md

This file provides guidance to Agents when working with this repository.

# IMPORTANT RULE

  - YOU MUST ONLY USE VALID ASCII UTF-8 LF CHARACTERS!

## Repository Overview

The **Ultimate AI Agent Ecosystem Directory 2025** is a data-driven documentation project that maintains a comprehensive, curated directory of AI agent frameworks, platforms, tools, and resources.

**CRITICAL:** This is NOT a software development project - there is no application code to build, test, or run. This is a documentation curation project using a modern data architecture:

- **YAML files** in `data/` are the single source of truth
- **README.md** is GENERATED - never edit directly
- **Static website** is GENERATED in `_site/` from YAML data
- Python scripts validate YAML and generate outputs

## Essential Build Commands

All development uses the Makefile:

```bash
make install    # Install Python dependencies in venv
make validate   # Validate all YAML files against schemas (ALWAYS RUN BEFORE COMMITTING)
make generate   # Generate README.md from YAML data
make site       # Generate static website in _site/
make serve      # Build site + start local server (http://localhost:8000)
make test       # Run validation + generation (CI-friendly)
make clean      # Remove generated files and cache
```

**Standard workflow:**
1. Edit YAML files in `data/agents/` or `data/categories/`
2. Run `make validate` to check schema compliance
3. Run `make generate` to update README.md
4. Run `make site` to regenerate website (optional)
5. Commit YAML files AND generated README.md

## Data Architecture

### Source of Truth: YAML Files

All content lives in structured YAML files:

**Agent/Tool Entries:** `data/agents/{category}/{name}.yml`

```yaml
# REQUIRED FIELDS
name: str                          # 1-100 characters
url: HttpUrl                       # Valid HTTP/HTTPS URL
description: str                   # 20-1000 characters
category: str                      # Must match a category ID

# CLASSIFICATION
type: framework|platform|tool|course|community|research
tags: List[str]                    # Lowercase, hyphenated

# OPTIONAL METADATA
subcategory: Optional[str]         # For grouping within categories
github_repo: Optional[str]         # Format: "owner/repo"
documentation_url: Optional[HttpUrl]
demo_url: Optional[HttpUrl]
platform: Optional[List[str]]      # e.g., ["Python", "TypeScript"]
license: Optional[str]             # e.g., "MIT", "Apache-2.0"
pricing: Optional[free|freemium|paid|enterprise]

# EDITORIAL FLAGS
featured: bool                     # Highlight on homepage
verified: bool                     # Link checked, complete metadata

# TRACKING
added_date: Optional[date]
last_verified: Optional[date]
```

**Category Definitions:** `data/categories/{category-id}.yml`

```yaml
id: str                            # URL-safe identifier
title: str                         # Display title
description: str                   # 10-500 characters
emoji: str                         # Default: "📦"
order: int                         # Display order (lower = earlier)
show_github_stats: bool            # Default: true
table_columns: List[str]           # Columns to show in tables
```

### Generated Outputs (DO NOT EDIT DIRECTLY)

1. **README.md** - Generated from `templates/readme.jinja2`
   - Markdown tables for GitHub display
   - Organized by categories with emoji headers
   - Badge with total entry count
   - **ALWAYS commit the generated README.md** (it's the main file users see on GitHub)

2. **_site/** - Static website (gitignored, not committed)
   - Interactive HTML pages with search/filtering
   - Generated from `templates/*.html.jinja2`
   - Deployed to GitHub Pages via Actions

### Data Flow

```
YAML Data (data/)
    ↓
Validation (scripts/validate.py using Pydantic schemas)
    ↓
Loading (load_categories() + load_agents())
    ↓
Template Rendering (Jinja2)
    ↓
Outputs (README.md + _site/)
```

## Key Files and Their Roles

### Data Files (Source of Truth)
- `data/agents/**/*.yml` - agent/tool entries organized by category
- `data/categories/*.yml` - category definitions

### Python Scripts
- `scripts/models.py` - Pydantic schemas (AgentEntry, Category, DirectoryMetadata)
- `scripts/validate.py` - YAML validation against schemas (exits with error on failure)
- `scripts/generate_readme.py` - README.md generator
- `scripts/generate_site.py` - Static website generator
- `scripts/migrate.py` - Migration tool (markdown → YAML, completed)

### Templates
- `templates/readme.jinja2` - README.md template
- `templates/base.html.jinja2` - Website base layout
- `templates/index.html.jinja2` - Homepage template
- `templates/category.html.jinja2` - Category page template

### Configuration
- `Makefile` - Build commands (primary interface)
- `requirements.txt` - Python dependencies (Pydantic, PyYAML, Jinja2)
- `.github/workflows/deploy.yml` - Auto-deploy to GitHub Pages on push to main
- `.github/workflows/validate.yml` - Validate YAML on PRs
- `.codex/prompts/*.md` - Shared Codex CLI prompt commands
- `.claude/commands/*.md` - Legacy Claude Code command compatibility

## Directory Structure

```
data/
├── agents/                        # YAML files (SOURCE OF TRUTH)
│   ├── open-source-frameworks/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moshehbenavraham/Ultimate-Agent-Directory](https://github.com/moshehbenavraham/Ultimate-Agent-Directory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
