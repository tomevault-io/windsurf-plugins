---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Name: Music Projects Website
Abbreviation: MPW

A static website generator that publishes information about all music projects. The site reads content from the [music-projects-database](../music-projects-database) workspace and renders it as a fully-featured HTML/CSS website with 40 projects.

The database contains 9 tables organized as Markdown files with YAML frontmatter. The build process generates a page for each project with 4 tabs:
- **Description** — Project overview and concert dates with locations
- **Schedule** — Rehearsals and concerts with location/address on the right and program details
- **Music** — Repertoire list with composer names and score links
- **Divisi** — Voice assignments in table format with composer names

## Directory Structure

```
music-projects-website/
├── CLAUDE.md              # This file
├── README.md              # Build instructions and deployment guide
├── scripts/               # Python build scripts
│   └── generate.py        # Python build script (no dependencies needed)
├── layout/                # Markdown layout templates (user-editable)
│   ├── index.md           # Homepage template
│   └── project.md         # Project detail page template
├── html/                  # Generated static website (output)
│   ├── index.html         # Homepage
│   ├── projects/          # Project detail pages
│   ├── css/style.css      # Responsive styling
│   └── ...                # Static assets
└── src/                   # (Legacy - not used, keep for reference)
```

## Build System

### Running the Build

```bash
python3 scripts/generate.py
```

This script:
1. Loads all database entries from `../music-projects-database`
2. Filters for Production projects (defined in `Production_PROJECTS` list)
3. Resolves relationships between tables via UUIDs
4. Extracts divisi information from repertoire markdown
5. Generates static HTML to `html/` folder

### Build Script Details

**Location**: `scripts/generate.py`

**Key Functions**:
- `parse_yaml()` — Parses YAML frontmatter without external dependencies
- `load_entries()` — Reads all markdown files from a table
- `get_project_agenda()` — Queries agenda items linked to a project
- `get_project_repertoire()` — Queries repertoire with music details
- `extract_divisi_html()` — Parses divisi tables from repertoire markdown
- `extract_program()` — Extracts program details from agenda markdown
- `generate_project_html()` — Renders project pages with all tabs

**Project Generation**:
The script automatically loads all projects from the database and generates a page for each. Projects are sorted alphabetically by title and assigned URL-safe filenames using the `slugify()` function.

## Working with the Database

### Data Flow

```
music-projects-database/
├── music-projects/ ──→ Project title, year, status, description
├── agenda/ ──────────→ Rehearsals/concerts with location_id, program
├── repertoire/ ──────→ Music pieces per project with divisi tables
├── music/ ───────────→ Piece titles, composers, score URLs
├── locations/ ───────→ Venue names, addresses
└── composers/ ───────→ Composer names
```

### Field References

**Agenda entries** have:
- `type` — "Rehearsal" or "Concert"
- `do_date` — ISO 8601 timestamp
- `location_id` — UUID reference to locations table
- `music_project_id` — UUID reference to music-projects table
- `## Program` section in markdown — Event description/program

**Repertoire entries** have:
- `order` — Sequence number
- `music_id` — UUID reference to music table
- `music_project_id` — UUID reference to music-projects table
- `## Divisi` section in markdown — Staff and voice assignments table

**Music entries** have:
- `music` — Title
- `composer_id` — UUID reference to composers table
- `voices` — SATB, SSATB, etc.
- `score_url` — Link to PDF

**Locations** have:
- `location` — Venue name
- `address` — Street address
- `city` — City name

### Key Principles

1. **UUID is the primary key** — All relationships use UUID, never text names
2. **YAML frontmatter is structured data** — Treat fields as queryable (like SQL columns)
3. **Markdown body has rich content** — ## Divisi and ## Program sections are parsed
4. **No external dependencies** — YAML parsing is done with regex/string operations
5. **Static output** — Generated HTML is fully independent, no build needed at runtime

## Page Rendering

### Index Page

Shows all Production projects in chronological order with:
- Project title
- Year
- Status badge (Completed/On Going/Cancelled)
- Excerpt
- "View Project" button

### Project Detail Pages

**Tab 1: Description**
- Project description text
- Concert dates with location names and addresses

**Tab 2: Schedule**
- List of all agenda items (rehearsals and concerts)
- Layout: Date | Details | Location (right side)
- Details: Type, time
- Program section from agenda markdown
- Location with address

**Tab 3: Music**
- Repertoire in order
- Each item: Number | Title | Composer | Score Link
- Score URL is a button linking to PDF

**Tab 4: Divisi**
- Each piece: Number, title, composer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [enrico-laboratory/music-projects-website](https://github.com/enrico-laboratory/music-projects-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
