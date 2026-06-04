---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a **Claude Skills marketplace plugin** that provides AI-native productivity skills for both Claude Code and Claude Desktop. The primary skill is **note-taking**, which transforms markdown notes into an AI-navigable "second brain" system.

**Key concept**: Skills make Claude an active partner in personal knowledge management rather than just a conversational assistant. They enable persistent memory, cross-project functionality, and natural interaction patterns.

## Project Architecture

### Directory Structure

```
productivity-skills/
├── .claude-plugin/
│   └── marketplace.json          # Plugin marketplace manifest
├── .github/
│   └── research/                 # Development research and analysis
├── plugins/
│   └── productivity-suite/       # Self-contained plugin bundle
│       └── skills/               # Production-ready skills
│           └── note-taking/      # Primary skill implementation
│               ├── SKILL.md      # Lean implementation guide (145 lines)
│               ├── scripts/
│               │   └── notes_manager.py  # Python utility for note operations
│               ├── templates/
│               │   └── monthly-template.md
│               └── examples/
│                   └── sample-notes.md   # Example note file
├── docs/                         # User documentation
│   ├── note-taking-guide.md      # Comprehensive user guide (383 lines)
│   ├── installation.md           # Installation and troubleshooting
│   ├── development.md            # Developer workflow
│   ├── contributing.md           # Contributing guidelines
│   └── faq.md                    # FAQ and troubleshooting
└── README.md                     # Concise overview (271 lines)
```

**Important:** This repository follows the Claude Code plugin marketplace pattern, NOT the simple skillDirectories approach. The root contains NO SKILL.md - individual skills are in `plugins/productivity-suite/skills/`.

### Note-Taking Skill Architecture

**Data Storage:**
- Notes stored in `~/Documents/notes/YYYY/MM-Month.md` by default
- Can be customized via `NOTES_DIR` environment variable
- All files are plain markdown for portability

**Core Components:**

1. **notes_manager.py**: Python utility that handles:
   - Adding new notes to monthly files
   - Searching across all notes with relevance scoring
   - Appending updates to existing entries
   - Index management (`.index.json`)
   - Statistics and analytics

2. **Relevance Scoring Algorithm** (in `calculate_relevance`):
   - File headers filtered out (e.g., "Notes - November 2025" not searchable)
   - Exact phrase match in heading: +500 points (overwhelming bonus)
   - All query terms in heading: +100 points
   - Individual terms in heading: +20 each
   - Terms in content: capped at +50 total (prevents content from overwhelming heading matches)
   - Recency boost: +10 (< 30 days), +5 (< 90 days), +2 (< 180 days)
   - Minimum relevance threshold: ≥50 required for updates (prevents weak matches)

3. **Entry Format**:
   ```markdown
   # Category - Brief description
   Content with multiple lines, code blocks, links, etc.

   **Created:** YYYY-MM-DD

   **Update (YYYY-MM-DD):** Additional information
   ```

   - New entries automatically get `**Created:** YYYY-MM-DD` timestamp
   - Updates automatically get `**Update (YYYY-MM-DD):**` timestamp

**Interaction Patterns:**

The skill responds to natural phrases:
- Adding: "Note that...", "Add a note about...", "Remember that..."
- Searching: "What did I note about...", "Status of...", "Find my notes on..."
- Updating: "Add to the X note...", "Update X with...", "Append to X..."

## Development Resources

### Research and Analysis Documents

**Location:** `.github/research/`

All development research, analysis, and summary documents should be placed in `.github/research/`. This keeps the repository clean for end users while preserving valuable context for contributors.

**Convention:** When conducting research for features, bug fixes, or architectural decisions:

1. Create comprehensive research/analysis documents in `.github/research/`
2. Use descriptive filenames: `research-<topic>.md`, `analysis-<issue>.md`, `summary-<topic>.md`
3. Include findings, trade-offs, recommendations, and references
4. Link to these documents from GitHub issues for context

**Current Research Documents:**
- `research-cross-platform-paths.md` - Cross-platform file path handling best practices
- `research-hooks-vs-utility-scripts.md` - Claude Code hooks vs utility scripts distinction
- `research-tiered-trigger-systems.md` - Natural language trigger system design
- `analysis-notes-manager-issues.md` - notes_manager.py issue analysis
- `summary-path-best-practices.md` - Quick reference for path handling

**Note:** GitHub issue descriptions are stored in GitHub Issues (#1, #2, etc.), not as markdown files in the repository.

## Development Commands

Since this is a skills plugin (not a traditional development project), there are no build/test commands. Testing is done through:

1. **Manual Testing in Claude**:
   ```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mcdow-webworks/productivity-skills](https://github.com/mcdow-webworks/productivity-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
