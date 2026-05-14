---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This repository contains a **Claude Code plugin marketplace** for the **Zettelkasten Flow** plugin, which transforms voice-transcribed notes into structured Zettelkasten notes within Obsidian. The plugin automates the workflow of processing voice captures from SuperWhisper into atomic, well-linked permanent notes.

## Plugin Architecture

### Plugin System Structure

```
diane/                              # Marketplace root
├── .claude-plugin/
│   └── marketplace.json            # Marketplace manifest
└── diane/                          # Plugin directory
    ├── .claude-plugin/
    │   └── plugin.json             # Plugin configuration
    ├── commands/                   # Slash command definitions
    │   ├── playback.md             # /playback command (voice note processor)
    │   ├── consult.md              # /consult command (vault analyst)
    │   └── find-links.md           # /find-links command (semantic linking)
    └── templates/                  # Obsidian Templater templates
        ├── permanent-note.md
        ├── fleeting-note.md
        ├── literature-note.md
        └── project-note.md
```

### Key Concepts

- **Marketplace**: A local plugin registry defined by `.claude-plugin/marketplace.json` at the repository root
- **Plugin**: A collection of slash commands, templates, and configuration stored in a subdirectory
- **Slash Commands**: Markdown files in `commands/` that define prompts Claude Code executes when invoked
- **Plugin Configuration**: JSON file defining vault paths, folder structure, and naming conventions

## Configuration Pattern

**IMPORTANT: This is an open-source plugin designed for use by multiple users.**

### Path Configuration

All directory paths are **configured during setup** via the `/diane:setup` command and stored in `diane/.claude-plugin/plugin.json`. Commands must **never hardcode paths** - they should always read from the plugin configuration.

**Configuration variables (in `plugin.json`):**
- `vault_path` - Absolute path to the user's Obsidian vault (empty by default, set during setup)
- `diane_folder` - Name of the Diane folder for voice captures
- `folders` - Object containing `clippings`, `fleeting`, `ideas`, `literature`, `permanent`, `project`, `output` folder names
- `naming` - Contains `style` (kebab-case) and `wikilink_format` (display-name)

### Obsidian Vault Structure

The plugin operates on any Obsidian vault configured by the user during setup.

**Default folder hierarchy:**
- `00 Diane/` - Voice note captures from SuperWhisper (timestamped files like `2025-10-13-0930.md`)
- `10 Clippings/` - Web captures from Obsidian browser extension (imported content, often work of others)
- `20 Fleeting notes/` - Quick captures, underdeveloped thoughts
- `30 Ideas/` - Sprawling proto-projects with many connections, not yet structured
- `40 Literature notes/` - Insights from books, articles, sources (processed clippings with your analysis)
- `50 Permanent notes/` - Atomic, well-developed ideas (highest value)
- `60 Project notes/` - Goal-oriented work, active projects
- `99 Output/` - Published work
- `_templates/` - Templater templates for note creation

**Note:** Users may customize these folder names during setup. Commands should reference the configured folder paths, not assume these defaults.

**Naming conventions:**
- Files: kebab-case (e.g., `ritual-interface-bridge.md`)
- Wikilinks: Display name format (e.g., `[[Ritual Interface Bridge]]`)

## Commands

### `/playback` - Voice Note Processor

**Purpose:** Process voice-transcribed notes from the Diane folder into structured Zettelkasten notes

**Workflow:**
1. Read plugin configuration to get vault path and folder structure
2. List unprocessed voice notes with preview from configured Diane and Fleeting folders
3. User selects a note to process
4. Read and analyze content:
   - Expand conversational transcription into clear prose
   - Fix transcription errors and complete thoughts
   - Identify atomic concepts (one idea per note)
   - Detect implicit references by checking recent notes
5. Find semantic connections across entire vault
6. Generate preview with suggested title, destination folder, and wikilinks
7. User approves, edits, skips, or cancels
8. Create note in destination folder, archive original to configured processed subfolder

**Key behaviors:**
- Interactive (one note at a time)
- Preserves user's writing voice
- Uses semantic search (not just keyword matching) for connections
- Suggests splitting multi-concept notes
- Resolves vague references ("that article") by temporal/semantic context

### `/find-links` - Semantic Link Discovery

**Purpose:** Find semantically related notes and suggest wikilinks for any note

**Workflow:**
1. User specifies target note (current file, path, or paste content)
2. Read and understand note's main concepts
3. Search entire vault for conceptual relationships:
   - Prioritize permanent notes (highest value)
   - Include literature notes for evidence
   - Connect to active projects
   - Flag orphaned fleeting notes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [popmechanic/diane](https://github.com/popmechanic/diane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
