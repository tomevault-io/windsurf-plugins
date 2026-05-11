---
trigger: always_on
description: **Goal:** Native macOS markdown editor for Astro content collections. Distraction-free writing environment with seamless frontmatter editing, inspired by iA Writer.
---

# Claude / AI Agent Instructions for Astro Editor

## Current Status

@CLAUDE.local.md

## Project Overview

**Goal:** Native macOS markdown editor for Astro content collections. Distraction-free writing environment with seamless frontmatter editing, inspired by iA Writer.

**Purpose:** Replace code editors for content writing. Focused environment understanding Astro's content structure.

**Key Features:**

- Auto-discovers Astro content collections from `src/content/config.ts`
- Dynamic frontmatter forms from Zod schemas
- File management with context menus
- Real-time auto-save every 2 seconds
- CodeMirror 6 with custom syntax highlighting
- Resizable panels, draft detection, file sorting
- Advanced editor features: URL clicking, drag & drop, markdown commands
- Comprehensive keyboard shortcuts and menu integration
- Toast notifications throughout the app

## Core Rules

### New Sessions

- Read @docs/TASKS.md for task management
- Review `docs/developer/architecture-guide.md` for essential patterns
- Consult specialized guides when working on specific features (see [Documentation Structure](#documentation-structure))
- Check git status and project structure

### Development Practices

**CRITICAL:** Follow these strictly:

1. **Read Before Editing**: Always read files first to understand context
2. **Follow Established Patterns**: Use patterns from this file and `docs/developer`
3. **Senior Architect Mindset**: Consider performance, maintainability, testability
4. **Batch Operations**: Use multiple tool calls in single responses
5. **Match Code Style**: Follow existing formatting and patterns
6. **Test Coverage**: Write comprehensive tests for business logic
7. **Quality Gates**: Run `pnpm run check:all` after significant changes
8. **No Dev Server**: Ask user to run and report back
9. **No Unsolicited Commits**: Only when explicitly requested
10. **Documentation**: Update `docs/developer/` guides for new patterns
11. **Removing files**: Always use `rm -f`

#### Directory Boundaries

- **Hooks belong in `/hooks/`**: If it exports a `use*` function, it goes in `/hooks/`
- **Pure functions in `/lib/`**: Business logic, utilities, classes
- **getState() is allowed**: One-way calls from lib to store using `getState()` are acceptable
- See `docs/developer/architecture-guide.md` for complete rules

### Documentation Structure

**Core Guides** (read for daily development):
- `docs/developer/architecture-guide.md` - Essential patterns and overview (START HERE)
- `docs/developer/state-management.md` - Deep dive into the "Onion" pattern and store decomposition
- `docs/developer/command-system.md` - Command pattern implementation and integration
- `docs/developer/ui-patterns.md` - Common UI patterns and shadcn/ui best practices
- `docs/developer/performance-patterns.md` - Performance optimization (getState, memoization)
- `docs/developer/testing.md` - Testing strategies and patterns

**System Documentation** (reference for system features):
- `docs/developer/cross-platform.md` - Platform detection, conditional compilation, title bar architecture
- `docs/developer/form-patterns.md` - Frontmatter fields and settings forms
- `docs/developer/schema-system.md` - Schema parsing and merging (Rust)
- `docs/developer/keyboard-shortcuts.md` - Implementing shortcuts
- `docs/developer/preferences-system.md` - Three-tier settings hierarchy
- `docs/developer/color-system.md` - Color tokens and dark mode
- `docs/developer/notifications.md` - Toast notification system
- `docs/developer/editor-styles.md` - CodeMirror syntax highlighting
- `docs/developer/recovery-system.md` - Crash recovery
- `docs/developer/logging.md` - Logging system

**Implementation** (optimization and build):
- `docs/developer/optimization.md` - Bundle optimization and performance budgets
- `docs/developer/releases.md` - Release workflow and process

**Feature Examples** (specific implementations):
- `docs/developer/feature-image-preview.md` - Image field preview implementation
- `docs/developer/astro-generated-contentcollection-schemas.md` - Astro JSON Schema reference

**Reference** (decisions and setup):
- `docs/developer/decisions.md` - Architectural decisions and trade-offs
- `docs/developer/apple-signing-setup.md` - Code signing and deployment

See `docs/README.md` for the complete categorized list.

### Documentation & Versions

- **Context7 First**: Always use Context7 for framework docs before WebSearch
- **Version Requirements**: Tauri v2.x, shadcn/ui v4.x, Tailwind v4.x, React 19.x, Zustand v5.x, CodeMirror v6.x, Vitest v4.x
- **Progress Tracking**: Update current task in `docs/tasks-todo` after major work

## Specialized Agents

The project has six specialized agents to help with complex implementation challenges:

### Project-Integrated Agents

**1. macos-ui-engineer** - Use when:

- Implementing native-feeling macOS UI patterns
- Working on typography, spacing, or visual hierarchy
- Creating or refining components that need to feel authentically Mac-like
- Applying Apple HIG principles to interface design

**2. react-performance-architect** - Use when:

- Reviewing React components for performance issues

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dannysmith/astro-editor](https://github.com/dannysmith/astro-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
