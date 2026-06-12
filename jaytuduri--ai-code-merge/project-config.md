---
trigger: always_on
description: **AI Code Merge** is a dual-interface tool that concatenates source code files into a single markdown document for AI analysis. It processes entire codebases by walking directory trees, filtering files based on patterns/exclusions, and aggregating content into structured markdown output.
---

# AI Code Merge - Developer Guide for AI Agents

## Project Overview

**AI Code Merge** is a dual-interface tool that concatenates source code files into a single markdown document for AI analysis. It processes entire codebases by walking directory trees, filtering files based on patterns/exclusions, and aggregating content into structured markdown output.

**Key Use Case**: Prepare codebases for input to LLMs by creating comprehensive source snapshots without build artifacts, dependencies, or IDE metadata.

## Architecture

### Package Structure (`src/aicodemerge/`)
- **`cli.py`**: CLI interface supporting local directories and Git URLs
- **`gui.py`**: PyQt5 GUI wrapper with drag-and-drop and Git URL support
- **`git_support.py`**: Isolated Git operations module
  - URL detection, validation, repository cloning
  - Git metadata capture (branch, commit, tags, status)
  - Binary file and LFS pointer detection
  - Accurate .gitignore handling with git check-ignore
- **`config.py`**: Shared configuration constants

### Two-Stage Analysis Pipeline

**Stage 1: Source Resolution** (`resolve_source()`)
- Detects if input is local path or Git URL (HTTPS, SSH, git://)
- For local: validates and returns directory
- For Git: clones to temp directory, captures metadata, returns working dir
- Returns `CloneContext` for proper cleanup management

**Stage 2: Analysis** (`run_analysis()`)
- Isolated pipeline accepting any directory
- 1. Parse configuration (depth, size, patterns)
- 2. Build directory structure tree
- 3. Collect files matching filters
- 4. Process files (detect binary, LFS, validate size)
- 5. Serialize output with Git metadata (if available)

### Core Processing Pipeline

1. **Path Traversal**: Recursive `os.walk()` respects depth limits via `current_depth` counter
2. **Intelligent Filtering**: Two-stage filtering using:
   - `.gitignore`-based patterns (dynamically parsed if exists)
   - Hardcoded `DEFAULT_EXCLUDE_PATTERNS` (310+ patterns covering Node, Python, Java, .NET, iOS, etc.)
   - Include-pattern matching (fnmatch) for `*.py`, `*.js`, etc.
   - Optional: git check-ignore for nested .gitignore accuracy
3. **Binary & LFS Detection** (NEW):
   - Binary: Null byte sampling in first 8KB
   - LFS: Header check for `version https://git-lfs.github.com/spec/v1`
4. **File Validation**: Size checks before appending (skips files >max_size_kb)
5. **Markdown Serialization**: Wraps file content in markdown code blocks with language detection

## New CLI Flags (Git Support)

```
Git Source & Clone Options:
  --branch BRANCH              Git branch to checkout (default: default branch)
  --commit HASH                Git commit hash to checkout
  --clone-depth N              Shallow clone depth (default: 1 for speed)
  --clone-timeout SEC          Clone operation timeout (default: 60 seconds)
  --submodules ignore|init     Submodule handling (default: ignore)
  --auth-token TOKEN           Auth token for private HTTPS repos (not recommended)
  --no-cleanup                 Keep cloned repo for inspection (temp dir location printed)

Advanced Filtering:
  --use-git-check-ignore       Use 'git check-ignore' for nested .gitignore accuracy
  --skip-binary                Skip files with null bytes (binary detection)
  --handle-lfs skip|pointer    LFS file handling (skip or show pointer)
```

All flags fully backward compatible with v1.0. Local analysis unaffected.

## Critical Implementation Details

### Shared Code Distribution
- **Single Source of Truth**: `src/aicodemerge/config.py` defines `DEFAULT_EXCLUDE_PATTERNS`, `DEFAULT_MAX_DEPTH`, `DEFAULT_MAX_SIZE`
- **Imported in Both**: CLI (`cli.py`) and GUI (`gui.py`) import from `aicodemerge.config`
- **Git Module**: `git_support.py` imported only when Git operations needed
- **Backward Compatible**: All v1.0 functionality unchanged, new features optional

### File Size Encoding
Files are read with `encoding="utf-8", errors="ignore"` to handle binary/non-UTF8 content gracefully (no crashes on encoding errors).

### Directory Listing Format
Tree structure uses ASCII art: `│   ` for indentation, `prefix + "│   "` for nested levels—purely cosmetic but hardcoded in two places.

### Git Metadata Capture (NEW)
Captured using subprocess `git` commands:
- Current branch: `git rev-parse --abbrev-ref HEAD`
- Commit hash: `git rev-parse HEAD`
- Short hash: First 7 characters
- Latest tag: `git describe --tags --abbrev=0`
- Dirty status: `git status --porcelain`
- All serialized in markdown header for reproducibility

### Binary & LFS Detection (NEW)
- **Binary Detection**: Sample first 8KB for null bytes
- **LFS Pointer Detection**: Check for `version https://git-lfs.github.com/spec/v1` header
- **Exclusion**: Both types excluded from content by default (configurable)

### Configuration Defaults
- **Max depth**: 4 (limits traversal; prevents deep node_modules exploration)
- **Max file size**: 100 KB (prevents huge outputs)
- **File patterns**: `*` (matches everything after exclusion filter)
- **Clone depth**: 1 (shallow, saves 50-90% bandwidth)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jaytuduri/ai-code-merge](https://github.com/jaytuduri/ai-code-merge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
