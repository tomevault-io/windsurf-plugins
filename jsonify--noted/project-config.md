---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

- When reporting information to me, be extremely concise and sacrifice grammar for the sake of concision.

## Quick Links

- **[notes.md](./notes.md)**: Complete list of implemented features and current capabilities
- **[plan.md](./plan.md)**: Development roadmap, planned features, and todo list
- **[docs/](./docs/)**: Always update the website documentation when new features are added

## Project Overview

This is a VS Code extension called "Noted" (published as "noted" by jsonify) that provides organized workspace notes with templates, search, and daily tracking. The extension creates a sidebar activity bar with a tree view for managing notes organized by year/month.

## Build & Development Commands

- **Install dependencies**: `pnpm install`
- **Compile TypeScript**: `pnpm run compile`
- **Watch mode for development**: `pnpm run watch`
- **Test extension**: Press F5 in VS Code to open Extension Development Host

## Architecture

### Core Components

**Modular Architecture** (completed):
The extension now uses a fully modular architecture with clear separation of concerns:

- **`src/extension.ts`** (1570 lines): Entry point and command registration
- **`src/constants.ts`**: Shared constants, templates, and patterns
- **`src/utils/`**: Validation and helper functions
  - `validators.ts`: Folder name validation
  - `dateHelpers.ts`: Date formatting utilities
  - `folderHelpers.ts`: Recursive folder operations
  - `frontmatterParser.ts`: YAML frontmatter parsing and tag extraction (v1.24.0)
  - `hierarchicalHelpers.ts`: Hierarchical note name parsing and validation (v1.45.0)
- **`src/services/`**: Business logic and file operations
  - `configService.ts`: Configuration management
  - `fileSystemService.ts`: Async file operation wrappers
  - `noteService.ts`: Note operations (search, stats, export)
  - `hierarchicalNoteService.ts`: Hierarchical note management with dot-delimited paths (v1.45.0)
  - `searchService.ts`: Advanced search with regex and filters (v1.6.0)
  - `templateService.ts`: Template generation
  - `tagService.ts`: Tag indexing and querying
  - `tagCompletionProvider.ts`: Tag autocomplete integration
  - `pinnedNotesService.ts`: Pinned notes management (v1.5.0)
  - `archiveService.ts`: Archive functionality (v1.5.0)
  - `linkService.ts`: Wiki-style links and backlinks (v1.5.0)
  - `connectionsService.ts`: Connection data for backlinks and outgoing links (v1.22.0)
  - `backlinksAppendService.ts`: Automatic backlinks sections appended to notes (v1.24.0)
  - `bulkOperationsService.ts`: Multi-select and bulk operations (v1.10.0)
  - `undoService.ts`: Undo/redo functionality (v1.13.0)
  - `undoHelpers.ts`: Undo operation helpers (v1.13.0)
  - `graphService.ts`: Graph data preparation and analysis (v1.14.0)
  - `activityService.ts`: Activity metrics collection and analysis (v1.36.0)
- **`src/search/`**: Smart Search with AI-powered semantic search (v1.40.0)
  - `types.ts`: TypeScript interfaces for search system
  - `QueryAnalyzer.ts`: Natural language query parsing and intent detection
  - `KeywordSearch.ts`: Enhanced keyword search with relevance scoring
  - `SemanticSearchEngine.ts`: AI-powered semantic search using VS Code LLM API
  - `SearchOrchestrator.ts`: Hybrid search combining keyword and semantic
- **`src/tagging/`**: Smart Auto-Tagging infrastructure (v1.40.0 - Phase 1)
  - `TagParser.ts`: YAML frontmatter tag parsing and writing
  - `TagManager.ts`: Tag CRUD operations, search, rename, merge
  - `TagGenerator.ts`: AI-powered tag generation using VS Code LLM API
  - `autoTagCommands.ts`: Command handlers for auto-tagging features
- **`src/templates/`**: Enhanced Template System (v1.41.0+ - Phases 1-4)
  - `TemplateTypes.ts`: TypeScript interfaces for templates and bundles (updated v1.43.3 with validation metadata)
  - `TemplateGenerator.ts`: AI-powered template generation and advanced validation (Phase 1, 3)
  - `BundleService.ts`: Multi-note workflow bundle creation and management (Phase 2)
  - `templateBrowserView.ts`: Visual template browser webview UI (Phase 4) with variable editor message handlers (Phase 3)
- **`src/providers/`**: VS Code tree view providers
  - `treeItems.ts`: Tree item classes (includes ConnectionSectionItem and ConnectionItem)
  - `templatesTreeProvider.ts`: Templates view
  - `notesTreeProvider.ts`: Main notes tree with drag-and-drop
  - `connectionsTreeProvider.ts`: Connections panel for showing backlinks and outgoing links (v1.22.0)
- **`src/commands/`**: Command handlers
  - `commands.ts`: Main command handlers
  - `tagCommands.ts`: Tag management commands
  - `bulkCommands.ts`: Bulk operation commands (v1.10.0)
  - `bundleCommands.ts`: Multi-note workflow bundle commands (v1.42.0)
  - `hierarchicalCommands.ts`: Hierarchical note creation and navigation (v1.45.0)
- **`src/calendar/`**: Calendar view functionality
  - `calendarHelpers.ts`: Calendar date operations
  - `calendarView.ts`: Webview and HTML generation
- **`src/graph/`**: Graph view visualization
  - `graphView.ts`: Interactive graph webview with vis.js
- **`src/activity/`**: Activity chart visualization (v1.36.0)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jsonify/noted](https://github.com/jsonify/noted) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
