---
trigger: always_on
description: This file contains context and memory for Claude AI when working on the clippy project.
---

# Claude AI Context File

This file contains context and memory for Claude AI when working on the clippy project.

## CRITICAL: Never Credit Yourself

**NEVER** add Claude credits to commits, code, or any project files. This includes:
- No "Generated with Claude Code" messages
- No "Co-Authored-By: Claude" in commits
- No mentions of AI assistance in code or documentation
- Keep all contributions attribution-free

## CRITICAL: Never Do Manual Release Steps

**NEVER** manually publish releases, edit releases, or do one-off fixes. The goal is ALWAYS a repeatable automated process. This includes:
- No `gh release edit` commands to fix drafts
- No manual edits to published releases
- No manual updates to Homebrew taps
- No workarounds - FIX THE AUTOMATION
- The goal is THE PROCESS, not getting a single release out

## Project Overview

Clippy is a macOS clipboard tool that bridges the gap between terminal file operations and GUI applications. It includes:

- **clippy**: Smart clipboard copying tool
- **pasty**: Intelligent clipboard pasting tool

## Recent Major Features

### Recent Downloads Functionality (v0.8.0+)
- Added `--recent` flag to clippy command (removed from pasty for better separation of concerns)
- Interactive picker using Bubble Tea library (replaced promptui for better multi-select support)
- Simplified `-r` behavior:
  - `-r` alone copies the most recent download (no picker)
  - `-r 3` copies the 3 most recent downloads
  - `-r 5m` copies all downloads from last 5 minutes
  - `-ri` shows interactive picker (can combine with numbers/durations)
- Multi-select support in picker: Space to toggle, Enter to copy, p to copy & paste
- Removed `--batch` flag (behavior integrated into numbered copies)
- Time-based filtering (e.g., `-r 5m`, `-r 1h`)
- macOS Downloads folder detection with smart archive handling
- Separate `--debug` and `--verbose` flags for better UX
- Config option `absolute_time = true` for absolute timestamps in picker

### Key Implementation Details

#### Library Structure
- `pkg/recent/recent.go`: Core library for recent downloads detection
- `pkg/recent/recent_test.go`: Comprehensive tests
- `internal/log/log.go`: Enhanced logging with debug support
- Library-first architecture with high-level business functions

#### Commands
- **clippy**: Uses `recent.GetRecentDownloads()` for core functionality, picker UI in cmd/
- **pasty**: No longer has recent downloads functionality (moved to clippy for cleaner separation)

#### Technical Features
- Cobra CLI framework for professional command-line interface
- Smart auto-unarchive detection for macOS Downloads folder
- Time-based filtering with duration parsing
- Batch handling for files downloaded together (within 30 seconds)
- Platform-specific build constraints (darwin vs windows)

## Development Guidelines

### Design Principles

#### Core vs Interface Philosophy (from Saša Jurić)
- **Core**: Implements the desired behavior of the system (what must be done regardless of how the system is accessed)
- **Interface**: Contains logic specific to how clients access the system (REST, CLI, GraphQL, etc.)
- **Key principle**: "Core implements behavior, Interface exposes it"
- **Decision rule**: If something is protocol-specific, it's an interface concern. If it must run in all cases, it's a core concern.
- **Benefits**: Clear separation allows developers to focus on one layer without understanding the other
- **CRITICAL**: Business logic NEVER goes in the interface layer!
  - Filtering files vs directories? Core concern - it's part of what "recent downloads" means
  - Excluding temp files? Core concern - it's part of the business rule
  - Converting user input to core types? Interface concern
  - Presenting data to users? Interface concern
- **Example violations to avoid**:
  - Don't filter data in controllers/views - the core should provide the right data
  - Don't validate business rules in the interface - the core enforces all rules
  - Don't make the interface "smart" - keep it dumb and focused on translation

#### Library-First Architecture
- **Core principle**: Implement all business logic as library functions first
- **Command-line tools**: Keep cmd/ tools as thin wrappers around library functions
- **Example**: `clippy.Copy()` function in library, `clippy` command calls it
- **Benefits**: Enables programmatic use, easier testing, cleaner separation of concerns
- **Pattern**: High-level business functions exposed through simple library APIs
- **UI in interface only**: Interactive elements (like pickers) belong in cmd/, not pkg/

#### CLI Design Philosophy
- **Professional CLI**: Use Cobra framework for consistent, professional command-line interface
- **Smart defaults**: Commands should work intuitively without excessive configuration
- **Composability**: Tools should work well together and with other Unix tools
- **Discoverability**: Use clear flag names and helpful examples

#### Code Organization
- `pkg/`: Public library packages (e.g., `pkg/clipboard`, `pkg/recent`)
- `internal/`: Private packages not meant for external use (e.g., `internal/log`)
- `cmd/`: Command-line applications as thin wrappers around library functions
- Build constraints for platform-specific code (e.g., `//go:build darwin`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neilberkman/clippy](https://github.com/neilberkman/clippy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
