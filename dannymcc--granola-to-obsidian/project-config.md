---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Granola Sync for Obsidian** is an Obsidian plugin that automatically syncs meeting notes from Granola AI to an Obsidian vault. It's a standalone Obsidian plugin (not a bundled/compiled project) delivered as three main files: `main.js`, `manifest.json`, and `styles.css`.

- **Type**: Obsidian plugin
- **Language**: JavaScript (Node.js)
- **Size**: `main.js` is ~2000 lines; all code in a single file
- **Version**: Currently 1.6.3
- **Minimum Obsidian Version**: 1.6.6

## Architecture & Structure

### Single-File Architecture
All plugin code lives in `main.js` (1994 lines) with two main classes:

1. **`GranolaSyncPlugin`** (extends `obsidian.Plugin`)
   - Main plugin class handling lifecycle, initialization, and sync orchestration
   - **Key methods**:
     - `onload()` - Plugin initialization, ribbon icon, settings tab
     - `syncNotes()` - Main sync orchestration loop (fetches documents, processes them, updates daily notes)
     - `loadCredentials()` - Reads Granola auth token from local filesystem
     - `fetchGranolaDocuments(token)` - API call to get meeting notes list
     - `fetchTranscript(token, docId)` - Optional transcript fetching
     - `processDocument(doc)` - Converts Granola document to Obsidian note with markdown
     - `updateDailyNote()` / `updatePeriodicNote()` - Integrates today's meetings into daily notes
     - Auto-sync setup and interval management

2. **`GranolaSyncSettingTab`** (extends `obsidian.PluginSettingTab`)
   - Settings UI panel with 25+ configurable options
   - **Key settings**:
     - `syncDirectory` - Where to save notes (default: "Granola")
     - `filenameTemplate` - Customizable note naming with tokens like `{title}`, `{created_date}`
     - `dateFormat` - Date formatting (YYYY-MM-DD, DD-MM-YYYY, etc.)
     - `autoSyncFrequency` - Auto-sync interval in milliseconds
     - `includeAttendeeTags` - Tag notes with meeting attendees
     - `skipExistingNotes` - Preserve manual edits in existing notes
     - `enableGranolaFolders` - Organize by Granola folder structure
     - `existingNoteSearchScope` - Search strategy for finding duplicates (sync dir, entire vault, or specific folders)

### Key Integration Points

**Granola API Integration**:
- Reads auth token from local Granola app (`~/Library/Application Support/Granola/supabase.json` on macOS)
- Makes authenticated API calls to fetch documents, transcripts, and folders
- Handles token formats: legacy supabase tokens and newer `workos_tokens` structure

**Obsidian Integration**:
- Uses Obsidian's Plugin API for file management, settings, status bar, commands
- Detects and integrates with Daily Notes and Periodic Notes plugins
- Converts Granola's ProseMirror format to markdown
- Frontmatter includes: `granola_id` (for duplicate detection), `created_at`, `updated_at`, `tags`, `granola_url`

**ProseMirror Conversion**:
- `proseMirrorToMarkdown()` converts Granola's rich editor format to markdown
- Handles headings, lists, bold, italic, links, code blocks, blockquotes

### Document Processing Flow

```
fetchGranolaDocuments() → [documents]
  ↓
for each document:
  → fetchTranscript() if enabled
  → processDocument()
    → proseMirrorToMarkdown() [content conversion]
    → generateFilename() [template + date formatting]
    → createOrUpdateNote() [file I/O]
    → extractAttendees() + apply tags [if enabled]
    → apply folder tags [if enabled]
  ↓
if enableDailyNoteIntegration:
  → updateDailyNote() [append today's meetings]
if enablePeriodicNoteIntegration:
  → updatePeriodicNote() [via Periodic Notes plugin]
```

## Development Commands & Building

**No build process**: This is not a bundled project. The plugin is delivered as raw JavaScript files that Obsidian loads directly.

**Files to distribute**:
- `main.js` - All plugin code (no transpilation)
- `manifest.json` - Plugin metadata
- `styles.css` - Plugin UI styles
- `versions.json` - Version history for Obsidian's plugin browser

## Release & Versioning Process

**Important**: This project has a strict, automated release workflow. See `.claude_rules` for full details.

### ⚠️ Critical Release Rules

**NEVER delete existing release tags or releases!** Users may need to downgrade to previous versions. If a release was created incorrectly:
- Create a NEW version with the next version number
- Do NOT delete and recreate the same version
- Example: If 1.7.1 exists and needs changes, create 1.7.2 instead

**ALWAYS update CHANGELOG.md before creating a release:**
- Add a new entry for the version at the top of CHANGELOG.md
- Include the date, features, fixes, and contributors
- Commit and push the CHANGELOG.md update
- Then create the version tag

**Author attribution:**
- All commits should be authored by the repository owner (dannymcc)
- The git config is already set up correctly
- **Do NOT include Co-Authored-By trailers** in commit messages
- Commit messages should be clean and professional without AI attribution

### Version Management
- **Format**: Semantic versioning without "v" prefix (e.g., `1.6.3`)
- Update both `manifest.json` and `versions.json` when bumping versions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dannymcc/Granola-to-Obsidian](https://github.com/dannymcc/Granola-to-Obsidian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
