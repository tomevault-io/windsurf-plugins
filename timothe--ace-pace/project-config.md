---
trigger: always_on
description: Ace-Pace development rules — tests, lint, git workflow, and project conventions
---


# Ace-Pace Project Rules

This file contains the development rules, guidelines, and technical reference for the Ace-Pace project. These rules should be followed by all AI agents working on this codebase.

## Core Workflow Requirements

When working on this project, you MUST:

1. **Always update tests when making changes**
   - Update existing tests if functionality changes
   - Add new tests for new features
   - **Always run tests and verify they pass** before completing work
   - Fix any failing tests
   - Run: `pytest` with coverage before completing work

2. **Always check for linter and SonarQube problems**
   - Run linter checks and fix any issues
   - Check SonarQube for code quality issues
   - Fix all identified problems

3. **Update documentation when appropriate**
   - Review README.md after significant changes
   - Update function documentation when signatures change
   - Keep technical documentation accurate

## Git Workflow Rules

**CRITICAL: Follow these git rules strictly:**

- **ABSOLUTELY NEVER run destructive git operations** (e.g., `git reset --hard`, `rm`, `git checkout`/`git restore` to an older commit) unless the user gives an explicit, written instruction. Treat these commands as catastrophic; if you are even slightly unsure, stop and ask before touching them.

- **Never use `git restore`** (or similar commands) to revert files you didn't author—coordinate with other agents instead so their in-progress work stays intact.

- **Always double-check git status** before any commit

- **Keep commits atomic**: commit only the files you touched and list each path explicitly
  - For tracked files: `git commit -m "<scoped message>" -- path/to/file1 path/to/file2`
  - For brand-new files: `git restore --staged :/ && git add "path/to/file1" "path/to/file2" && git commit -m "<scoped message>" -- path/to/file1 path/to/file2

- **Quote any git paths** containing brackets or parentheses (e.g., `src/app/[candidate]/**`) when staging or committing so the shell does not treat them as globs or subshells.

- **When running `git rebase`**, avoid opening editors—export `GIT_EDITOR=:` and `GIT_SEQUENCE_EDITOR=:` (or pass `--no-edit`) so the default messages are used automatically.

- **Never amend commits** unless you have explicit written approval in the task thread.

- **Delete unused or obsolete files** when your changes make them irrelevant (refactors, feature removals, etc.), and revert files only when the change is yours or explicitly requested.

- **Before attempting to delete a file** to resolve a local type/lint failure, stop and ask the user. Other agents are often editing adjacent files; deleting their work to silence an error is never acceptable without explicit approval.

- **NEVER edit `.env`** or any environment variable files—only the user may change them.

- **Coordinate with other agents** before removing their in-progress edits—don't revert or delete work you didn't author unless everyone agrees.

- **Moving/renaming and restoring files** is allowed.

## Code Quality Standards

- Follow PEP 8 Python style guide
- Maintain cognitive complexity ≤ 15 per function
- Use descriptive variable names
- Use docstrings for functions
- Keep functions focused and single-purpose
- Use `_` prefix for private/internal helper functions
- Comprehensive test suite exists in `tests/` directory (100+ tests)
- Ensure all tests pass before completing work

## Project Overview

**Ace-Pace** is a Python-based tool designed to help users manage and organize their One-Pace anime library. It automates:
- Identifying which One-Pace episodes are already in the user's local library
- Detecting missing episodes
- Automatically downloading missing episodes via BitTorrent clients
- Renaming local files to match official One-Pace naming conventions
- Maintaining a database of episode metadata and file checksums

## Core Functionality

### Episode Discovery and Indexing
- Scrapes Nyaa.si torrent tracker for One-Pace episodes
- Extracts CRC32 checksums from episode filenames or torrent file lists
- **Quality Filtering**: Only extracts episodes with 1080p quality
  - Episodes without quality markers are excluded
  - Episodes with quality other than 1080p are excluded
  - Quality filtering is applied in both `fetch_episodes_metadata()` and `fetch_crc32_links()`
  - Filtering is case-insensitive (accepts 1080P, etc.)
- **URL Parameter Support**: Both `fetch_episodes_metadata()` and `update_episodes_index_db()` accept a `base_url` parameter
  - Quality filtering still applies regardless of URL parameters
- Builds and maintains an episodes index database (`episodes_index.db`)
- Supports both single-file and multi-file torrent structures
- Handles pagination to fetch all available episodes

### Local Library Management
- Scans local directories recursively for video files (`.mkv`, `.mp4`, `.avi`)
- Calculates CRC32 checksums for local video files
- **Path Normalization**: All file paths are normalized before storage and lookup
  - Uses `normalize_file_path()` to resolve symlinks and convert to absolute paths
  - Ensures consistent path representation across different OS and environments
  - Prevents cache misses when same file is accessed via different path representations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [timothe/Ace-Pace](https://github.com/timothe/Ace-Pace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
