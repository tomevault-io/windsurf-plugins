---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a high-performance CLI tool that migrates Notion exports to Obsidian-compatible markdown format. The tool is written in JavaScript and uses Bun as the runtime.

## Commands

### Running the migration

```bash
# Basic migration (current directory)
./notion2obsidian.js

# Single zip file
./notion2obsidian.js ./Export-abc123.zip

# Multiple zip files with glob patterns
./notion2obsidian.js *.zip
./notion2obsidian.js Export-*.zip

# Multiple zip files with custom output
./notion2obsidian.js *.zip -o ~/Obsidian/Notion-Import

# Directory processing with output
./notion2obsidian.js ./my-notion-export -o ~/Documents/Obsidian

# Using bun directly
bun run notion2obsidian.js ./my-export

# Using npm scripts
bun run migrate ./my-export
bun run dry-run ./my-export
```

### Command line options

- `-o, --output DIR` - Output directory for processed files (default: extract location)
- `-d, --dry-run` - Preview changes without modifying files (extracts 10% sample or 10MB max for zip files)
- `-v, --verbose` - Show detailed processing information
- `-h, --help` - Show help message
- `-V, --version` - Show version number
- `--no-callouts` - Disable Notion callout conversion to Obsidian callouts
- `--no-csv` - Disable CSV database processing and index generation
- `--dataview` - Create individual MD files from CSV rows (default: keep CSV only)
- `--no-banners` - Disable cover image detection and banner frontmatter

### Testing

```bash
# Run tests
bun test

# Watch mode
bun test --watch
```

### Dependencies

```bash
# Install dependencies
bun install
```

## Architecture

### Modular Structure (v2.4.0+)

The tool is organized into focused modules for maintainability and AI-context-friendliness:

**Core Libraries** (`src/lib/`):
- `utils.js` (88 lines) - Shared utilities and regex patterns (PATTERNS, BATCH_SIZE)
- `stats.js` (37 lines) - Migration statistics tracking (MigrationStats class)
- `cli.js` (134 lines) - Command-line argument parsing and help text
- `links.js` (99 lines) - Markdown to wiki-link conversion and file mapping
- `callouts.js` (113 lines) - Notion callout transformation to Obsidian format
- `frontmatter.js` (341 lines) - YAML frontmatter generation and metadata extraction
- `scanner.js` (96 lines) - File and directory traversal with glob patterns
- `assets.js` (66 lines) - User interaction and directory operations
- `zip.js` (371 lines) - Archive extraction and merging with fflate
- `csv.js` (275 lines) - Database processing and Dataview integration
- `enrich.js` (735 lines) - Notion API enrichment functionality (experimental)

**Main Entry Point:**
- `notion2obsidian.js` (1,146 lines) - Runtime check, main migration logic, CLI routing

**Benefits of Modular Design:**
- Each module < 400 lines (fits in AI context windows)
- Clear separation of concerns
- Easy to locate and modify functionality
- Fully tested (94 tests passing)
- Backward compatible

### Core Processing Pipeline

The migration happens in two phases:

**Phase 0: Zip Extraction (if needed)**

- Detects `.zip` files and extracts to same directory as zip
- Creates shortened directory names: `Export-2d6f-extracted/` (first 4 chars of hash instead of full UUID)
- Uses pure JS `fflate` library for reliable extraction (handles special characters)
- Filters out macOS metadata files (`__MACOSX`, hidden files)
- Automatically identifies single top-level directory after extraction
- Shows extracted directory location and `rm -rf` command for cleanup after migration

**Phase 1: Analysis & Planning**

1. Scans directory for all `.md` files and directories using `Glob`
2. Builds a file map for link resolution (handles URL-encoded names)
3. Extracts metadata (Notion IDs, folder structure, tags)
4. Detects duplicate filenames across different folders
5. Generates preview of changes with estimated link conversion count (samples 10 files)

**Phase 2: Execution**

1. **Content Processing**: Adds frontmatter and converts markdown links to wiki links (batch processed, 50 files at a time)
2. **File Renaming**: Removes Notion IDs from filenames
3. **Directory Renaming**: Removes Notion IDs from folder names (processed deepest-first to avoid path conflicts)

### Key Patterns & Optimizations

**Regex Patterns** (defined in `PATTERNS` object at top of script):

- `hexId`: Matches 32-character hexadecimal Notion IDs
- `mdLink`: Matches markdown links `[text](file.md)`
- `frontmatter`: Detects existing frontmatter (handles BOM and whitespace)
- `notionIdExtract`: Extracts Notion ID from filename

**Performance Features**:

- Single file read per file (combines metadata extraction and content processing)
- Batch processing with `Promise.all()` (default: 50 files at a time, configurable via `BATCH_SIZE` constant)
- Pre-compiled regex patterns
- `Map` data structures for O(1) file lookups
- Sampling technique to estimate total link count (processes 10 files to calculate average)

**Duplicate Handling**:

- Files with identical names after ID removal are tracked in `duplicates` Map
- Folder paths stored in frontmatter `folder` field for disambiguation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bitbonsai/notion2obsidian](https://github.com/bitbonsai/notion2obsidian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
