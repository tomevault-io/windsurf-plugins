---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**django-countries** is a Django application that provides country choices for use with forms, flag icons static files, and a country field for models. It provides all ISO 3166-1 countries as choices with support for translations via Django's gettext.

## Development Commands

This project uses **uv** (fast package manager) and **just** (command runner). See `docs/contributing.md` for setup instructions.

### Testing
```bash
# Run all test environments + coverage (recommended)
just test

# Quick test with current Python (no coverage matrix)
just test quick

# Run specific test environment
just test [latest|previous|legacy|latest-pyuca|latest-noi18n]

# Examples:
just test latest           # Python 3.14 + Django 6.0
just test previous         # Python 3.13 + Django 5.2
just test legacy           # Python 3.10 + Django 4.2
just test latest-pyuca     # With Unicode collation
just test latest-noi18n    # Without i18n

# Run specific environment with custom Python version
just test [latest|previous|legacy|latest-pyuca|latest-noi18n] [3.10-3.14]

# Example:
just test latest 3.14      # Latest Django with Python 3.14

# Run a single test file
uv run --group test pytest django_countries/tests/test_fields.py

# Run a specific test
uv run --group test pytest django_countries/tests/test_fields.py::TestCountryField::test_name
```

### Code Quality
```bash
# Run ALL checks (format, lint, type, docs, tests)
just check

# Run individual tools directly
uv run ruff format django_countries           # Format code
uv run ruff check django_countries            # Lint code
uv run mypy django_countries                  # Type check
uv run bandit -r django_countries -x tests    # Security scan
```

### Coverage
Coverage is automatically generated when running `just test`. View the HTML report at `htmlcov/index.html`.

### Documentation
```bash
just docs         # Serve documentation locally at http://127.0.0.1:8080
just deploy_docs  # Build and deploy documentation to GitHub Pages
```

Documentation is built with MkDocs. Use `just deploy_docs` to manually deploy to GitHub Pages, or it will be automatically deployed during `just deploy`.

**When adding new features to documentation**, mark them with version callouts using the `!!! info` admonition format:

```markdown
!!! info "New in development version"

Full description of the feature continues here...
```

For subsection-level features (like specific setting options), use an indented brief description inside the admonition, then continue with full documentation outside:

```markdown
!!! info "New in development version"

    Brief description of what's specifically new about this option.

Full documentation of the option continues here with examples, code blocks, etc.
```

Example:
```markdown
!!! info "flag_url: New in development version"

    Per-country `flag_url` overrides allow custom country codes to reuse existing flag image assets.

A custom flag image URL for this country. Usage examples:
...
```

During release, the deployment script automatically replaces all "New in development version" markers with "New in version X.Y.Z" based on the release version. This ensures users know which version introduced each feature.

### Updating Country Data

Country data should be manually updated from the official ISO 3166-1 Online Browsing Platform (OBP):

1. Visit: https://www.iso.org/obp/ui/
2. Click the 'Country Codes' radio button
3. Click the search button (🔍)
4. Change 'Results per page' to 300
5. Select and copy the table data
6. Paste into a spreadsheet (LibreOffice Calc, Excel, etc.)
7. Verify columns: Country Name, Alpha-2, Alpha-3, Numeric
8. Delete any extra columns (like French names)
9. Delete the header row
10. Save as `django_countries/iso3166-1.csv`
11. Run: `uv run --group dev python django_countries/data.py` to regenerate `data.py`

The official OBP data uses specific formatting (parentheses vs commas) documented in `docs/iso3166-formatting.md`.

### Translation Commands (for Maintainers)
```bash
# Update English source file with new translatable strings (after editing data.py or base.py)
just tx-makemessages

# Pull and compile translations from Transifex
just tx-pull
```

**Translation Workflow**:
1. When country names change in `data.py` or `base.py`, run `just tx-makemessages`
2. This generates/updates `django_countries/locale/en/LC_MESSAGES/django.po` (English source)
3. Commit the English source file
4. Push to Transifex with `tx push -s` (done automatically in deploy)
5. Translators update translations on Transifex
6. Pull translations with `just tx-pull`

### Release Commands (for Maintainers)
```bash
# Deploy a release to PyPI (fully automated)
just deploy          # Interactive prompt for version bump
just deploy patch    # For bug fixes (7.7.0 -> 7.7.1)
just deploy minor    # For new features (7.7.0 -> 7.8.0)
just deploy major    # For breaking changes (7.7.0 -> 8.0.0)

# Dry-run mode to preview and validate changes
DRY_RUN=1 just deploy
DRY_RUN=1 just deploy patch

# Allow uncommitted changes (not recommended)
just deploy patch --allow-dirty

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SmileyChris/django-countries](https://github.com/SmileyChris/django-countries) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
