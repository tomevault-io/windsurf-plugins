---
trigger: always_on
description: This is an Obsidian plugin that provides Electronic Lab Notebook (ELN) functionality. The plugin is written in TypeScript and uses modern build tools.
---

# Obsidian ELN Plugin - Copilot Instructions

This is an Obsidian plugin that provides Electronic Lab Notebook (ELN) functionality. The plugin is written in TypeScript and uses modern build tools.

## Project Structure and File Placement Guidelines

### Source Code Organization

- **`src/`** - All TypeScript source code
  - `src/main.ts` - Plugin entry point
  - `src/commands/` - Command implementations
  - `src/core/` - Core business logic and services
  - `src/data/` - Data models and persistence
    - `src/data/templates/` - **CANONICAL SOURCE** for all template structures
      - Contains actual TypeScript template objects used by the plugin
      - Always reference these files, not documentation examples
      - Templates are stored as TypeScript exports with "display" parameter
  - `src/events/` - Event handling and system events
  - `src/search/` - Search functionality
  - `src/settings/` - Plugin settings and configuration
  - `src/styles/` - SCSS/CSS styles (compiled to styles.css)
  - `src/types/` - TypeScript type definitions and interfaces
  - `src/ui/` - User interface components and views
  - `src/utils/` - Utility functions and helpers

### Testing

- **`tests/`** - All test files go here
  - Use descriptive names like `test-[feature-name].ts`
  - Include unit tests, integration tests, and validation scripts
  - Test data and fixtures can be placed in `tests/memory/` or similar subdirectories

### Documentation

- **`docs/`** - All documentation files
  - `docs/user/` - User-facing documentation (installation, features, guides)
    - **Published on GitHub Pages** - Visible to all users
  - `docs/examples/` - Example templates and usage examples
    - **Published on GitHub Pages** - Visible to all users
  - `docs/developer/` - Developer documentation (API, architecture, setup)
    - `docs/developer/public/` - **PUBLIC DEVELOPER DOCS** (Published on GitHub Pages)
      - Contains: README.md, ROADMAP.md, KNOWN-ISSUES.md
      - Visible to users and contributors on documentation site
    - All other `docs/developer/` subfolders are **INTERNAL ONLY**
      - Available in git repository for contributors
      - Hidden from GitHub Pages (not published on documentation site)
  - `docs/archive/` - Historical documentation (internal only)
  - `docs/_layouts/` - Jekyll templates for GitHub Pages
  - `docs/assets/` - CSS and other assets for GitHub Pages

### Developer Documentation Organization (CRITICAL)

The `docs/developer/` directory has a **two-tier organizational structure**:

#### Public Developer Documentation (Published on GitHub Pages)

**`docs/developer/public/`** - **PUBLICLY VISIBLE ON DOCUMENTATION SITE**
- `public/README.md` - Developer documentation index and contributing guide
- `public/ROADMAP.md` - Project roadmap and version planning
- `public/KNOWN-ISSUES.md` - Known bugs and limitations
- `public/index.md` - Landing page for developer section

**Purpose**: Information useful for users, potential contributors, and the community
**Visibility**: Published at https://fcskit.github.io/obsidian-eln-plugin/developer/public/

#### Internal Developer Documentation (Hidden from GitHub Pages)

All other `docs/developer/` subfolders are **INTERNAL ONLY**:
- Available in the git repository for active contributors
- Hidden from GitHub Pages (excluded in `docs/_config.yml`)
- Visible to anyone who clones the repository

**`docs/developer/todos/`** - **PRIMARY TASK TRACKING SYSTEM**
- `todos/active/` - Current work in progress
- `todos/completed/` - Finished features and improvements
- `todos/planned/` - Future features and improvements

**Use this for ALL task tracking.** When working on features:
1. Check `todos/active/` for current priorities
2. Create detailed plans in `todos/planned/` for future work
3. Move completed work to `todos/completed/` with full documentation
4. **ALWAYS update public/ROADMAP.md** when adding/completing todos

**`docs/developer/template-system/`** - Template redesign documentation
- All template-related design docs, proposals, implementation guides
- Has its own README.md index

**`docs/developer/guides/`** - Testing, debugging, release workflows
- Testing guides, release checklists, debugging procedures
- Has its own README.md index

**`docs/developer/note-creation-architecture/`** - Note creation system design
- Architecture documentation for planned redesign
- **Note**: This is PLANNED work, not current implementation

**`docs/developer/core/`** - Core system documentation
- Architecture, API reference, development setup

**`docs/developer/components/`** - Component-specific docs

**`docs/developer/infrastructure/`** - Build system, logging, CSS

**`docs/developer/contributing/`** - Contribution guidelines

**`docs/developer/archive/`** - Historical documentation
- Completed phase reports, old debugging docs, outdated tracking
- Move docs here when they're no longer actively maintained

## TODO System Workflow (CRITICAL)

### Primary Reference for Work Planning

**ALWAYS check `docs/developer/todos/` and `docs/developer/public/ROADMAP.md` before starting work.**

The todo system is the single source of truth for:
- What needs to be done (`todos/planned/`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fcskit/obsidian-eln-plugin](https://github.com/fcskit/obsidian-eln-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
