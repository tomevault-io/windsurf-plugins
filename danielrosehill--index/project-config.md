---
trigger: always_on
description: This repository is an automated indexing system for Daniel's GitHub repositories. It pulls public repositories, categorizes them by topic and time, and generates comprehensive index files for easy browsing.
---

# CLAUDE.md - GitHub Master Index Repository

## Project Overview

This repository is an automated indexing system for Daniel's GitHub repositories. It pulls public repositories, categorizes them by topic and time, and generates comprehensive index files for easy browsing.

## Purpose

The main goals of this system are:
- Automatically discover and categorize new GitHub repositories
- Maintain organized index files by topic and creation date
- Generate comprehensive README files with repository statistics
- Provide multiple views of the repository collection (by-topic, by-time, main index)

## Repository Structure

```
.
├── scripts/             # All indexing and generation scripts
│   ├── sync-all.sh                   # Master sync script (runs everything)
│   ├── pull-and-index.py             # Pulls repos & auto-categorizes
│   ├── generate-index.py             # Generates index.md
│   ├── update-time-indexes.py        # Updates time-based indexes
│   ├── build-hierarchical-readme.py  # Builds main README
│   ├── generate-category-indexes.py  # Generates category indexes
│   ├── hierarchy-schema.json         # Category hierarchy and keywords
│   └── run-*.sh                      # Individual script wrappers
├── sections/            # Organized repository sections
│   ├── by-topic/       # Topical categorization
│   │   ├── ai-ml/      # AI & Machine Learning
│   │   ├── data-tools/ # Data processing tools
│   │   ├── development/# Development tools
│   │   └── ...
│   └── by-time/        # Chronological organization (by creation date)
│       ├── 2025/       # Year directories
│       │   └── 01_25.md, 02_25.md, etc.
│       └── README.md   # Time index overview
├── repo-data/          # Cached repository data from GitHub
│   ├── all-repos-YYYYMMDD-HHMMSS.json  # Timestamped snapshots
│   └── latest.json     # Symlink to most recent data
├── index.md            # Main index (all repos by update date)
└── README.md           # Main readme with navigation
```

## Quick Start

### Complete Sync (Recommended)
Run the master sync script to update everything:
```bash
./scripts/sync-all.sh
```

This runs all steps in order:
1. Pulls latest repos from GitHub
2. Auto-categorizes new repos into sections
3. Generates main index.md
4. Updates time-based indexes
5. Generates category indexes
6. Builds README.md

### Individual Scripts

If you need to run specific steps:

```bash
# 1. Pull and categorize new repos only
./scripts/run-indexer.sh

# 2. Generate main index only
./scripts/run-index-generator.sh
# or with fresh GitHub data:
python3 scripts/generate-index.py --refresh

# 3. Update time indexes only
python3 scripts/update-time-indexes.py

# 4. Build README only
python3 scripts/build-hierarchical-readme.py

# 5. Generate category indexes only
python3 scripts/generate-category-indexes.py
```

## Key Scripts

### 1. sync-all.sh (Master Script)
**Location:** `scripts/sync-all.sh`

Runs the complete workflow in correct order. Use this for most updates.

### 2. pull-and-index.py
**Location:** `scripts/pull-and-index.py`

- Pulls all public repos using `gh` CLI
- Scans existing section files to find indexed repos
- Auto-categorizes new repos based on keywords from `hierarchy-schema.json`
- Generates indexing report showing:
  - High confidence matches (auto-added)
  - Low confidence matches (need manual review)

### 3. generate-index.py
**Location:** `scripts/generate-index.py`

- Creates `index.md` with ALL repos sorted by update date (newest first)
- Includes: description, stars, forks, topics, dates
- Use `--refresh` flag to pull fresh GitHub data first

### 4. update-time-indexes.py
**Location:** `scripts/update-time-indexes.py`

- Creates/updates chronological organization in `sections/by-time/`
- Organizes by CREATION date (not update date)
- Creates year directories with monthly files (01_25.md, 02_25.md, etc.)
- Updates year and main time index READMEs

### 5. build-hierarchical-readme.py
**Location:** `scripts/build-hierarchical-readme.py`

- Generates the main `README.md` from `hierarchy-schema.json`
- Creates category structure with badges
- Adds navigation links

### 6. generate-category-indexes.py
**Location:** `scripts/generate-category-indexes.py`

- Creates `index.md` files for each category directory
- Lists subcategories and their files

## Categorization System

Categories are defined in `scripts/hierarchy-schema.json`:

**Main Categories:**
- **AI & Machine Learning** - AI agents, LLM tools, prompt engineering
- **Data Tools** - Data processing, analysis, visualization
- **Development** - Code generation, IDEs, GitHub tools
- **Infrastructure** - Automation, backups, Linux tools
- **Platforms & Services** - Platform-specific integrations
- **Tools & Utilities** - General CLI/GUI utilities
- **Project Types** - Templates, experiments, awesome lists, misc

Each category has:
- **Keywords**: Used for auto-categorization matching
- **Subsections**: Nested organization
- **Files**: Individual `.md` files for each subcategory

## Typical Workflows

### Full Sync (Most Common)
```bash
# Run complete sync
./scripts/sync-all.sh

# Review changes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danielrosehill/Index](https://github.com/danielrosehill/Index) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
