---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## CRITICAL INFORMATION FOR CLAUDE

**YOU WROTE THIS ENTIRE CODEBASE (100%)** - The user (lefticus) has only provided guidance and direction. You are responsible for all code, tests, documentation, and GitHub issue management. Remember this between sessions.

**ALWAYS TAKE RESPONSIBILITY FOR FAILING TESTS** - All tests pass at the start of every session unless the user explicitly states otherwise. If tests fail during your work, it is YOUR bug to fix. Do not try to make problems "not your problem."

## Repository Overview

This is a Pandoc-first tool for converting C++ draft standard LaTeX sources from the [cplusplus/draft](https://github.com/cplusplus/draft) repository to high-quality GitHub Flavored Markdown. The tool is production-ready with comprehensive test coverage.

## Quick Start

**ALWAYS run this script at the start of every development session:**

```bash
./setup-and-build.sh
```

This is the **standard entry point** for all development work. The script:
1. Creates/activates local `venv` if needed
2. Installs/updates Python dependencies (smart timestamp checking - skips if up-to-date)
3. Verifies Pandoc 3.0+ is available
4. Clones/updates `cplusplus-draft` repository into project directory
5. Runs full test suite in parallel (**aborts on any test failure**)
6. Converts n4950 (C++23) to markdown in `n4950/` directory

**Why use this script:**
- ✅ Idempotent - safe to run repeatedly (skips unnecessary work)
- ✅ Fast when already set up (checks timestamps, only updates what changed)
- ✅ Works offline (if cplusplus-draft already exists)
- ✅ Guarantees tests pass before proceeding
- ✅ Ensures consistent environment across all development sessions

**Do not skip this step.** Even if you think everything is set up, run it to verify the environment is correct.

## Development Setup (Manual)

If you need manual control instead of using `setup-and-build.sh`:

```bash
# Create and activate virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Clone cplusplus/draft repository (into project directory)
git clone https://github.com/cplusplus/draft.git cplusplus-draft
```

**Note:** The package is **not installed** - all scripts and tests run directly from the source directory. This eliminates confusion between installed and source files.

## Testing

**Note:** `./setup-and-build.sh` runs the full test suite automatically. Use these commands for manual testing during development:

```bash
# Run all tests
./venv/bin/pytest tests/ -v

# Run specific test module
./venv/bin/pytest tests/test_filters/test_code_blocks.py -v
./venv/bin/pytest tests/test_filters/test_macros.py -v
./venv/bin/pytest tests/test_integration/test_chapters.py -v

# Run with coverage report
./venv/bin/pytest tests/ --cov=src/cpp_std_converter --cov-report=html

# Run tests in parallel (faster for full suite)
./venv/bin/pytest tests/ -v -n auto
```

**Test Structure:**
- `tests/test_filters/` - Unit tests for individual Lua filters
- `tests/test_integration/` - Integration tests using actual C++ standard files
- Other tests - Converter, repo manager, stable names, utils, etc.
- Integration tests use **n4950 (C++23)** as the stable baseline
- Tests look for `cplusplus-draft/` in the project directory (or fall back to `/tmp/cplusplus-draft/`)

## CLI Usage

All tools run directly from the repository root without installation. The `./setup-and-build.sh` script
sets up dependencies and runs the full conversion pipeline automatically.

**Main tools:**
- `./convert.py` - Convert C++ draft LaTeX to Markdown (wrapper for `src/cpp_std_converter/converter.py:main`)
- `./generate_diffs.py` - Generate diffs between C++ standard versions
- `./generate_html_site.py` - Generate HTML site from diffs

```bash
# Convert LaTeX to Markdown
./convert.py intro.tex -o intro.md
./convert.py --build-separate -o n4950/ --git-ref n4950
./convert.py --build-full -o full.md --git-ref n4950
./convert.py --list-tags

# Generate diffs between versions
./generate_diffs.py n3337 n4950
./generate_diffs.py --list

# Generate HTML site from diffs
./generate_html_site.py --output build/site/
./generate_html_site.py --test
```

## Architecture

### Three-Stage Pipeline

1. **Minimal Preprocessing** (Python) - Repository management and file discovery
2. **Conversion** (Pandoc + Lua Filters) - **Heavy lifting** happens here: LaTeX parsing, AST transformations, Markdown generation
3. **Minimal Post-processing** (Python) - Output format generation and metadata

### Core Components

- `src/cpp_std_converter/converter.py` - Main conversion logic + CLI (entry point)
- `src/cpp_std_converter/repo_manager.py` - Git operations for cplusplus/draft repo
- `src/cpp_std_converter/standard_builder.py` - Builds full/separate standard documents
- `src/cpp_std_converter/stable_name.py` - Extracts stable names from `\rSec0` tags
- `src/cpp_std_converter/label_indexer.py` - Cross-reference link management
- `src/cpp_std_converter/filters/*.lua` - Pandoc Lua filters for LaTeX transformations

### Lua Filter Pipeline


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lefticus/cppstdmd](https://github.com/lefticus/cppstdmd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
