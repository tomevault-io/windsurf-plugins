---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

F2 Commander is an orthodox (two-panel) file manager for the terminal, built with modern Python. It extends the traditional file manager concept to work with local files, remote storage systems (S3, FTP, SFTP, etc.), and archives as navigable filesystems.

**Key Technologies:**
- **Textual**: Modern TUI framework for terminal applications
- **fsspec**: Universal filesystem abstraction for local/remote/archive access
- **libarchive-c**: Reading and writing numerous archive formats
- **Pydantic**: Type-safe configuration management
- **Rich**: Terminal text rendering and syntax highlighting

**Project Structure:**
```
f2/
├── app.py              # Main application and orchestration
├── main.py             # Entry point and CLI
├── config.py           # Configuration management with Pydantic
├── commands.py         # Command data structures
├── errors.py           # Error handling decorators
├── shell.py            # External tool detection and integration
├── update.py           # Update checking
├── fs/
│   ├── node.py         # Immutable filesystem node abstraction
│   ├── util.py         # Low-level filesystem operations
│   └── arch.py         # Archive filesystem support
├── widgets/
│   ├── filelist.py     # File browser widget (main UI)
│   ├── preview.py      # File preview widget
│   ├── panel.py        # Panel container (can host different widget types)
│   ├── dialogs.py      # Dialog widgets
│   ├── bookmarks.py    # Bookmark management
│   ├── config.py       # Configuration dialog
│   ├── connect.py      # Remote connection dialog
│   ├── form.py         # Form helpers
│   └── help.py         # Help widget
└── tcss/
    └── main.tcss       # Textual CSS styling
```

## Documentation Structure

This documentation is organized into three categories:

### Product Requirements Documents (PRD)
**What the product does** - User-facing features and functionality

- **[prd-file-management.md](doc/prd-file-management.md)**: Core file operations (copy, move, delete, rename, navigation, selection, sorting, search)
- **[prd-remote-filesystems.md](doc/prd-remote-filesystems.md)**: Remote storage access (S3, FTP, SFTP, cloud drives, specialized systems)
- **[prd-archives.md](doc/prd-archives.md)**: Archive support (browsing, extracting, creating archives in various formats)
- **[prd-preview.md](doc/prd-preview.md)**: File preview panel (text with syntax highlighting, images, PDFs, directory trees)
- **[prd-configuration.md](doc/prd-configuration.md)**: Configuration system (settings, bookmarks, remote connections, external tools)

### Architecture Documents (ARCH)
**How the system is designed** - Technical architecture and design decisions

- **[arch-ui-framework.md](doc/arch-ui-framework.md)**: Textual TUI framework integration (reactive components, message passing, styling, async operations)
- **[arch-filesystem-abstraction.md](doc/arch-filesystem-abstraction.md)**: fsspec filesystem abstraction (Node design, protocol support, metadata normalization, operations)
- **[arch-config-management.md](doc/arch-config-management.md)**: Pydantic configuration (validation, autosave, platform-specific paths, migration)
- **[arch-external-tools.md](doc/arch-external-tools.md)**: External tool integration (editor, viewer, shell detection, subprocess management, OS integration)

### Standard Operating Procedures (SOP)
**How to implement features** - Implementation patterns and guidelines

- **[sop-debugging.md](doc/sop-debugging.md)**: Debugging and bug fixing (analyzing data flow, common bug patterns, debugging strategy, testing methodology)
- **[sop-error-handling.md](doc/sop-error-handling.md)**: Error handling patterns (decorators, context managers, dialog display, logging)
- **[sop-adding-features.md](doc/sop-adding-features.md)**: Adding new features (actions, dialogs, file operations, configuration, panel types, FileList modifications)
- **[sop-working-with-nodes.md](doc/sop-working-with-nodes.md)**: Node abstraction usage (creation, navigation, operations, patterns, pitfalls)

## When to Use Each Document Type

### Starting a New Feature?
1. Check **PRD** documents to understand existing related features
2. Review **ARCH** documents for the subsystems you'll work with
3. Follow **SOP** documents for implementation patterns

### Understanding Existing Code?
1. Start with **PRD** to understand what the feature does
2. Read **ARCH** to understand the underlying design
3. Reference **SOP** for common patterns used

### Debugging an Issue?
1. Check **SOP** debugging guide for systematic approach
2. Review **PRD** for expected behavior
3. Check **ARCH** for architectural constraints
4. Reference **SOP** for correct implementation patterns

### Refactoring?
1. **PRD**: Ensure functionality is preserved
2. **ARCH**: Understand design constraints and dependencies
3. **SOP**: Update patterns if they change

## Quick Reference

### Key Classes and Their Locations

| Class | Location | Purpose |
|-------|----------|---------|
| `F2Commander` | `f2/app.py:84` | Main application, orchestrates everything |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [candidtim/f2-commander](https://github.com/candidtim/f2-commander) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
