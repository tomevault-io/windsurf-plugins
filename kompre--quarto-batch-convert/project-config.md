---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`quarto-batch-convert` is a CLI tool that batch converts Jupyter notebooks (.ipynb) to Quarto documents (.qmd) and vice versa. It wraps the `quarto convert` command to process multiple files at once, with support for regex-based filtering, filename transformations, and directory structure preservation.

## Development Commands

### Setup
```bash
# Sync dependencies from uv.lock (run at start of session)
uv sync
```

### Testing
```bash
# Run all tests
uv run pytest tests -v

# Run a specific test file
uv run pytest tests/test_quarto_batch_convert.py -v

# Run a specific test function
uv run pytest tests/test_quarto_batch_convert.py::test_match_and_replace_pattern -v
```

### Running the CLI locally
```bash
# Install in development mode
uv sync

# Run the tool
uv run quarto-batch-convert <args>
# or using the alias
uv run qbc <args>
```

### Building
```bash
# Build the package
uv build
```

## Code Architecture

### Core Module Structure

- **`src/quarto_batch_convert/quarto_batch_convert.py`**: Main module containing all CLI logic
  - `convert_files()`: Entry point CLI command (click-based) that orchestrates the conversion process
  - `convert_file()`: Worker function that converts individual files via subprocess calls to `quarto convert`
  - `collect_files_from_directory()`: Gathers files from directories with optional recursion
  - `check_quarto_installation()`: Validates Quarto CLI is available

### Key Design Patterns

1. **Parallel Processing**: Uses `concurrent.futures.ProcessPoolExecutor` to convert multiple files concurrently (quarto_batch_convert.py:284-302)

2. **Regex-based Filtering and Transformation**:
   - Match patterns filter which files to process
   - Replace patterns transform output filenames via `re.sub()`
   - Pattern format: `MATCH/REPLACE` or just `MATCH`

3. **Directory Structure Preservation**: Uses `os.path.relpath()` to maintain source directory hierarchy in output (quarto_batch_convert.py:106-108)

4. **Dual-direction Conversion**: The `--qmd-to-ipynb` flag toggles conversion direction by swapping input/output extensions

### Testing Approach

Tests use a module-scoped fixture (`setup_teardown_test_env`) that creates a temporary directory structure with sample files. Tests invoke the CLI using `click.testing.CliRunner` and verify file creation/transformation.

Key test coverage areas:
- Regex match/replace patterns
- Recursive vs non-recursive directory traversal
- Prefix and extension handling
- Output path management
- Directory structure preservation

## External Dependencies

- **Quarto CLI**: Required external dependency (not Python). The tool checks for it via `shutil.which("quarto")` and exits with instructions if missing.
- **Click**: CLI framework for argument parsing and command structure
- **pytest**: Test framework

## Dependency Management

- Use `uv` for all dependency management operations
- Never modify `pyproject.toml` directly - use `uv add`, `uv remove`, etc.
- The project uses `uv_build` as the build backend (pyproject.toml:30-31)

## Version Management

Version is stored in `src/quarto_batch_convert/version.py` and used by the CLI's `--version` flag.

## Repository Labels

### Workflow Trigger Labels
- `code-review` - Trigger automated Claude code review on PR
- `release` - Publish package to PyPI after merge
- `test-release` - Publish package to TestPyPI after merge

### Version Bump Labels
Applied to PRs to indicate semantic version increment:
- `bump:major` - Breaking changes (YYYY.0.0)
- `bump:minor` - New features (YYYY.M.0)
- `bump:patch` - Bug fixes (YYYY.M.D)
- `bump:stable` - Remove pre-release suffix
- `bump:alpha`, `bump:beta`, `bump:rc`, `bump:post`, `bump:dev` - Pre-release versions

## Release Workflow

The project uses an automated release workflow triggered by PR labels. See `.github/RELEASE.md` for complete documentation.

### Quick Release Guide

1. **Create PR** targeting `main` with your changes
2. **Add version bump label(s)**:
   - `bump:patch` - Bug fixes (2025.8.11 → 2025.8.12)
   - `bump:minor` - New features (2025.8.11 → 2025.9.0)
   - `bump:major` - Breaking changes (2025.8.11 → 2026.0.0)
   - Multiple labels allowed (applied sequentially)
3. **Add release type label**:
   - `release` - Publish to PyPI (production)
   - `test-release` - Publish to TestPyPI (testing only)
4. **Wait for checks** to pass (test + version-bump workflows)
5. **Merge PR** - Automatic publishing happens after merge

### Workflow Files

- `.github/workflows/version-bump.yml` - Automatically bumps version in PR based on labels
- `.github/workflows/release.yml` - Builds, tests, publishes package, creates tag and GitHub Release
- `.github/workflows/test.yml` - Runs tests on PRs (required check)

### Post-merge Actions

After merging a PR with release labels:
- Package built and published to PyPI/TestPyPI
- Git tag created (e.g., `v2025.9.0`)
- GitHub Release created with PR reference
- Test releases marked as pre-release

### Configuration Notes

- **PyPI Trusted Publishing** must be configured for both `pypi` and `testpypi` environments

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kompre) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
