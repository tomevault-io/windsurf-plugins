---
trigger: always_on
description: **GitHub repository**: `markuskiller/vba-edit`
---

# Repository

**GitHub repository**: `markuskiller/vba-edit`
Always use this exact owner/name for all `gh` CLI calls — never guess or derive it.

# Project Overview

This project is a command-line application that allows users to export, import, and edit VBA code from Microsoft Office files using external editors with one-way sync back to Office.

## Current Features (v0.4.x)

**Status**: Active refinement across multiple 0.4.x releases

**Supported Applications:**
- ✅ Microsoft Excel (VBA: modules, classes, forms)
- ✅ Microsoft Word (VBA: modules, classes, forms)
- ✅ Microsoft Access (VBA: modules, classes only - NO forms)
- ✅ Microsoft PowerPoint (VBA: modules, classes, forms)

**Core Functionality:**
- **Export**: One-time export of VBA code to `.bas`, `.cls`, `.frm` files
- **Import**: One-time import of VBA code from files back to Office
- **Edit**: Live editing with **one-way sync** (external editor → Office VBA)
  - Initial export on command start
  - Watches filesystem for changes
  - Syncs changes back to Office VBA on save [CTRL+S]
  - **NO bi-directional sync**: Changes made in Office VBA editor while edit mode is active are NOT synced back to files

**Advanced Features (Being Refined):**
- RubberduckVBA folder structure support (refining)
- Configuration file support with TOML (optimizing)
- Colorized CLI output with uv-style aesthetics (implemented in v0.4.1)
- Safety features and data loss prevention (refining)
- Enhanced CLI with organized help and grouped options (refining)
- Windows binaries with security verification (ongoing)

## Planned Features (Future)

**Not Yet Implemented:**
- ❌ PowerQuery support (M language) - v0.5.0 (next release)
- ❌ Bi-directional sync (Office VBA → external editor) - v0.5.0 via keyboard shortcut approach
- ❌ Code signing for Windows binaries (SignPath.io) - v0.6.0+ if feasible

**Detailed Roadmap**: See `docs/ROADMAP.md` (private, not published to GitHub during alpha/beta)

**Note**: No specific date commitments - features released when ready

## Folder Structure

- `/src`: Contains the source code
- `/docs`: Contains documentation for the project, including ideas for further enhancements and even programming guides for beginners.

## General Instructions

The project's language is English.

Write code (names of classes, objects, variables, methods, comments etc) always in the project language.

Also express all your answers in project language, even when prompted in any other language.
As an exception, the developer may ask for a translation of terms and sentences he or she doesn't understand, on his/her behalf.

---

## Development Environment Setup

### CRITICAL: Always Set Up Environment First

**Before running any Python commands, tests, or scripts:**

1. **Sync dependencies** (ensures all packages including dev dependencies are installed):
   ```powershell
   uv sync --extra dev
   ```

2. **For Python commands** - Use one of these methods:
   ```powershell
   # Method 1: Use uv run (recommended - handles environment automatically)
   uv run python script.py
   uv run pytest tests/
   uv run ruff check
   
   # Method 2: Activate virtual environment manually
   .venv\Scripts\Activate.ps1  # PowerShell
   python script.py
   pytest tests/
   ```

3. **Common commands**:
   ```powershell
   # Install/update dependencies
   uv sync --extra dev
   
   # Run tests (ALWAYS use -v for verbose output, NEVER pipe to see progress)
   uv run pytest -v
   uv run pytest tests/test_cli_common.py -v
   
   # IMPORTANT: Never pipe pytest output - user wants to see test progress in real-time
   # DON'T DO: uv run pytest -v | something
   # DO: uv run pytest -v (shows progress as tests run)
   
   # Code quality
   uv run ruff check --fix
   uv run ruff format
   
   # Run CLI tools
   uv run python -m vba_edit.word_vba --help
   ```

### Why This Matters

- **Missing dependencies**: Without `uv sync`, pytest, ruff, and other dev tools won't be available
- **Import errors**: Python modules won't be found if environment isn't activated
- **CI/CD parity**: Using `uv run` or activated venv matches CI/CD behavior
- **Rich dependency**: Colorization features require `rich>=13.0.0` to be installed

### Troubleshooting

If you see errors like:
- `ModuleNotFoundError: No module named 'pytest'` → Run `uv sync --extra dev`
- `ModuleNotFoundError: No module named 'rich'` → Run `uv sync`
- `Command 'pytest' not found` → Use `uv run pytest` or activate venv first

---

## Version Control

Currently, some developers are new to working with Git, Github and Copilot.
When working on a feature, suggest which branch to use.

When the developer asks questions about tools he uses, switch to a different chat reserved for these questions.

Microsoft Copilot suggested the branching strategy outlined in .\docs\development\BRANCHING.md

## Github Copilot Modes

When in Agent mode, don't modify any code, unless you did check out to a new Git branch before.
Each prompt shall use the context @workspace automatically.

## CHANGELOG Guidelines

When updating CHANGELOG.md:

### Read First
- **Always read the entire CHANGELOG thoroughly** before making changes
- Check for duplicates - don't repeat existing entries
- Understand the context of what's already documented

### Writing Style

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [markuskiller/vba-edit](https://github.com/markuskiller/vba-edit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
