---
trigger: always_on
description: Generates monthly and yearly statistics reports in HTML, TXT (WeeWX-style ASCII), and JSON formats:
---

# AGENTS.md - MeshCore Stats Project Guide

> **Maintenance Note**: This file should always reflect the current state of the project. When making changes to the codebase (adding features, changing architecture, modifying configuration), update this document accordingly. Keep it accurate and comprehensive for future reference.

## Important: Source Files vs Generated Output

**NEVER edit files in `out/` directly** - they are generated and will be overwritten.

| Type | Source Location | Generated Output |
|------|-----------------|------------------|
| CSS | `src/meshmon/templates/styles.css` | `out/styles.css` |
| HTML templates | `src/meshmon/templates/*.html` | `out/*.html` |
| JavaScript | `src/meshmon/templates/*.js` | `out/*.js` |

Always edit the source templates, then regenerate with `python scripts/render_site.py`.

## Running Commands

**IMPORTANT: Always activate the virtual environment before running any Python commands.**

```bash
cd /path/to/meshcore-stats
source .venv/bin/activate
python scripts/render_site.py
```

Configuration is automatically loaded from `meshcore.conf` (if it exists). Environment variables always take precedence over the config file.

## Development Workflow

### Test-Driven Development (TDD)

**MANDATORY: Always write tests BEFORE implementing functionality.**

When implementing new features or fixing bugs, follow this workflow:

1. **Write the test first**
   - Create test cases that define the expected behavior
   - Tests should fail initially (red phase)
   - Cover happy path, edge cases, and error conditions

2. **Implement the minimum code to pass**
   - Write only enough code to make tests pass (green phase)
   - Don't over-engineer or add unrequested features

3. **Refactor if needed**
   - Clean up code while keeping tests green
   - Extract common patterns, improve naming

Example workflow for adding a new function:

```python
# Step 1: Write the test first (tests/unit/test_battery.py)
def test_voltage_to_percentage_at_full_charge():
    """4.20V should return 100%."""
    assert voltage_to_percentage(4.20) == 100.0

def test_voltage_to_percentage_at_empty():
    """3.00V should return 0%."""
    assert voltage_to_percentage(3.00) == 0.0

# Step 2: Run tests - they should FAIL
# Step 3: Implement the function to make tests pass
# Step 4: Run tests again - they should PASS
```

### Pre-Commit Requirements

**MANDATORY: Before committing ANY changes, run lint, type check, and tests.**

```bash
# Always run these commands before committing:
source .venv/bin/activate

# 1. Run linter (must pass with no errors)
ruff check src/ tests/ scripts/

# 2. Run type checker (must pass with no errors)
python -m mypy src/meshmon --ignore-missing-imports

# 3. Run test suite (must pass)
python -m pytest tests/ -q

# 4. Only then commit
git add . && git commit -m "..."
```

If lint, type check, or tests fail:
1. Fix all lint errors before committing
2. Fix all type errors before committing - use proper fixes, not `# type: ignore`
3. Fix all failing tests before committing
4. Never commit with `--no-verify` or skip checks

### Running Tests

```bash
# Run all tests
python -m pytest tests/

# Run with coverage report
python -m pytest tests/ --cov=src/meshmon --cov-report=term-missing

# Run specific test file
python -m pytest tests/unit/test_battery.py

# Run specific test
python -m pytest tests/unit/test_battery.py::test_voltage_to_percentage_at_full_charge

# Run tests matching a pattern
python -m pytest tests/ -k "battery"

# Run with verbose output
python -m pytest tests/ -v
```

### Test Organization

```
tests/
├── conftest.py              # Root fixtures (clean_env, tmp dirs, sample data)
├── unit/                    # Unit tests (isolated, fast)
│   ├── test_battery.py
│   ├── test_metrics.py
│   └── ...
├── database/                # Database tests (use temp SQLite)
│   ├── conftest.py          # DB-specific fixtures
│   └── test_db_*.py
├── integration/             # Integration tests (multiple components)
│   └── test_*_pipeline.py
├── charts/                  # Chart rendering tests
│   ├── conftest.py          # SVG normalization, themes
│   └── test_chart_*.py
└── snapshots/               # Golden files for snapshot testing
    ├── svg/                 # Reference SVG charts
    └── txt/                 # Reference TXT reports
```

### Coverage Requirements

- **Minimum coverage: 95%** (enforced in CI)
- Coverage is measured against `src/meshmon/`
- Run `python -m pytest tests/ --cov=src/meshmon --cov-fail-under=95`

## Commit Message Guidelines

This project uses [Conventional Commits](https://www.conventionalcommits.org/) with [release-please](https://github.com/googleapis/release-please) for automated releases. **Commit messages directly control versioning and changelog generation.**

### Format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Commit Types

| Type | Description | Version Bump | Changelog Section |
|------|-------------|--------------|-------------------|
| `feat` | New feature or capability | Minor (0.1.0 → 0.2.0) | Features |
| `fix` | Bug fix | Patch (0.1.0 → 0.1.1) | Bug Fixes |
| `perf` | Performance improvement | Patch | Performance Improvements |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jorijn/meshcore-stats](https://github.com/jorijn/meshcore-stats) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
