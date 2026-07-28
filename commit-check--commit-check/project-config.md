---
trigger: always_on
description: Commit Check is a Python CLI tool and pre-commit hook that validates commit messages, branch naming, author information, and more according to Conventional Commits and Conventional Branch conventions.
---

# Commit Check Development Instructions

Commit Check is a Python CLI tool and pre-commit hook that validates commit messages, branch naming, author information, and more according to Conventional Commits and Conventional Branch conventions.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Architecture (v2.0)

### Core Design Principles
**v2.0** is a complete rewrite following SOLID principles with TOML-based configuration (replacing YAML from v1.x).

**Configuration Priority Cascade** (highest to lowest):
1. CLI arguments (`--subject-max-length=72`)
2. Environment variables (`CCHK_SUBJECT_MAX_LENGTH=72`)
3. TOML configuration files (`cchk.toml`, `commit-check.toml`, `.github/cchk.toml`, `.github/commit-check.toml`)
4. Built-in defaults (see `commit_check/__init__.py`)

### Module Responsibilities

```
commit_check/
├── main.py              # CLI entry point, argument parsing
├── config_merger.py     # Merges CLI→Env→TOML→Defaults
├── config.py           # TOML file loading logic
├── rule_builder.py     # Builds ValidationRule objects from config
├── rules_catalog.py    # Catalog of all validation rules (COMMIT_RULES, BRANCH_RULES)
├── engine.py           # ValidationEngine orchestrates BaseValidator subclasses
├── imperatives.py      # Imperative verb list for subject validation
└── util.py             # Git operations, output formatting, helpers
```

**Key Flow**: `main()` → `ConfigMerger` → `RuleBuilder` → `ValidationEngine` → `BaseValidator` subclasses → exit code 0/1

### Configuration Discovery
Tool searches for TOML files in this order:
1. Path from `--config` argument
2. `cchk.toml` in current directory
3. `commit-check.toml` in current directory
4. `.github/cchk.toml`
5. `.github/commit-check.toml`

### Pre-commit Integration Patterns
- **File argument**: `commit-check -m .git/COMMIT_EDITMSG` (from pre-commit hook)
- **Stdin piped**: `echo "feat: add feature" | commit-check -m`
- **Git commands**: Uses `subprocess.run()` to invoke git for branch names, author info, etc.

## Working Effectively

### Prerequisites and Setup
```bash
# Install Python dependencies (required)
python3 -m pip install --upgrade pip
python3 -m pip install nox

# Install package in development mode with PYTHONPATH (network timeout workaround)
export PYTHONPATH=/home/runner/work/commit-check/commit-check
python3 -m pip install pyyaml  # Core dependency
```

### Build and Package
```bash
# Build wheel package -- NETWORK ISSUES: Often fails due to PyPI timeouts in CI environments
# Takes ~10 seconds when working, ~20+ seconds when timing out
nox -s build

# Manual build workaround when nox fails:
python3 -m pip wheel --no-deps -w dist .  # NETWORK ISSUES: Also fails due to build dependencies

# Install wheel (depends on build)
nox -s install  # NETWORK ISSUES: Often fails due to PyPI timeouts in CI environments
```

### Testing
```bash
# Run tests directly (fastest, most reliable) -- takes ~1 second. Set timeout to 30+ seconds.
PYTHONPATH=/home/runner/work/commit-check/commit-check python3 -m pytest tests/ -v

# Alternative: Run tests via nox (may timeout due to network)
nox -s coverage  # NETWORK ISSUES: Often fails due to PyPI timeouts, takes 5+ minutes when working
```

### Linting and Code Quality
```bash
# NETWORK ISSUES: nox -s lint often fails due to pre-commit installation timeouts
# Manual workaround for linting:
python3 -m pip install pre-commit  # May timeout, retry if needed
pre-commit install --hook-type pre-commit
pre-commit run --all-files --show-diff-on-failure  # Takes 2-5 minutes for full run
```

### Documentation
```bash
# NETWORK ISSUES: Documentation builds fail due to external dependencies (fonts.google.com)
# Install docs dependencies (may timeout)
python3 -m pip install sphinx-immaterial sphinx-autobuild

# Build docs -- FAILS due to network restrictions in CI environments
PYTHONPATH=/home/runner/work/commit-check/commit-check sphinx-build -E -W -b html docs _build/html
```

## Validation Scenarios

### Manual Testing (Essential after changes)
Always manually validate functionality with these scenarios:

```bash
# Set PYTHONPATH for direct module execution
export PYTHONPATH=/home/runner/work/commit-check/commit-check

# Test 1: Valid conventional commit message
echo "fix: correct issue with validation" > test_commit.txt
python3 -m commit_check.main --message test_commit.txt
# Expected: Exit code 0 (success)

# Test 2: Invalid commit message format
echo "invalid commit message" > test_commit_invalid.txt
python3 -m commit_check.main --message test_commit_invalid.txt
# Expected: Exit code 1, shows ASCII art rejection and error details

# Test 3: Complex conventional commit with scope and body
echo "feat(api): add new user endpoint

This adds support for creating new users via the REST API.

Breaking Change: API version updated to v2" > test_complex_commit.txt
python3 -m commit_check.main --message test_complex_commit.txt
# Expected: Exit code 0 (success)

# Test 4: Branch name validation
python3 -m commit_check.main --branch

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [commit-check/commit-check](https://github.com/commit-check/commit-check) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
