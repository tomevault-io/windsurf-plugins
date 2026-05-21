---
trigger: always_on
description: **Stellium** is a modern, extensible Python library for computational astrology built on Swiss Ephemeris for NASA-grade astronomical accuracy.
---

# Claude Development Instructions for Stellium

**Stellium** is a modern, extensible Python library for computational astrology built on Swiss Ephemeris for NASA-grade astronomical accuracy.

---

## Table of Contents

- [Environment Setup Requirements](#environment-setup-requirements)
- [Codebase Architecture](#codebase-architecture)
- [Directory Structure](#directory-structure)
- [Core Principles](#core-principles)
- [Development Workflows](#development-workflows)
- [Testing Conventions](#testing-conventions)
- [Code Style & Conventions](#code-style--conventions)
- [Common Tasks](#common-tasks)
- [Important Patterns](#important-patterns)
- [Anti-Patterns to Avoid](#anti-patterns-to-avoid)
- [Quick Reference](#quick-reference)

---

## Environment Setup Requirements

**CRITICAL: Always run these commands before executing any Python code that uses Swiss Ephemeris:**

```bash
source ~/.zshrc
pyenv activate starlight
```

### Why This Matters

The Swiss Ephemeris dependency (`pyswisseph`) requires specific environment setup:
- The `starlight` pyenv environment contains the correct Python version (3.11+) and dependencies
- Swiss Ephemeris data files are configured for this specific environment in `data/swisseph/ephe/`
- Without proper activation, imports will fail or calculations will be incorrect

### Required Environment Commands

**Before running Python files:**
```bash
source ~/.zshrc && pyenv activate starlight && python [file]
```

**Before running tests:**
```bash
source ~/.zshrc && pyenv activate starlight && pytest
source ~/.zshrc && pyenv activate starlight && python tests/test_chart_generation.py
source ~/.zshrc && pyenv activate starlight && python tests/moon_phase_tester.py
```

**Before running examples:**
```bash
source ~/.zshrc && pyenv activate starlight && python examples/usage.py
source ~/.zshrc && pyenv activate starlight && python examples/viz_examples.py
```

### Development Setup

```bash
# Install in development mode with all dev dependencies
pip install -e ".[dev]"

# Set up pre-commit hooks (automatic code formatting)
pre-commit install

# Run all tests
pytest

# Run with coverage
pytest --cov=src --cov-report=term-missing
```

---

## Documentation Organization

All project documentation lives under `docs/` with the following structure:

- **`docs/planning/`** -- Future work, designs not yet started. Put new feature plans here.
- **`docs/development/`** -- Active development guides, architecture references, implementation guides. Put technical references and in-progress implementation docs here.
- **`docs/archive/`** -- Fully implemented plans, kept for historical reference. When a planning doc is fully implemented, move it here and add an `> **ARCHIVED**` notice at the top.
- **`docs/`** (root) -- User-facing guides (visualization, reports, chart types, etc.)
- **`claude_info/`** -- Research and analysis documents (competitive analysis, codebase audit, novel features)

**Index:** See `docs/DOCS_INDEX.md` for a complete list of all documentation with descriptions and status.

**When adding new docs:**
- Planning a new feature? Create `docs/planning/YOUR_FEATURE.md`
- Writing a dev guide or reference? Create `docs/development/YOUR_GUIDE.md`
- Finished implementing a plan? Move it from `planning/` to `archive/` with an archive notice

---

## Codebase Architecture

Stellium is built on three foundational principles that enable extensibility, maintainability, and performance:

### 1. **Protocols over Inheritance**
- Uses structural typing (Protocols) instead of class hierarchies
- Enables "duck typing" with type safety
- Components implement interfaces without inheritance
- Easy to test and compose

### 2. **Composability**
- All components work independently and can be freely combined
- Builder pattern for fluent configuration
- Lazy evaluation (configure first, calculate when ready)
- Mix-and-match engines and components

### 3. **Immutability**
- All data models are frozen dataclasses (cannot be modified after creation)
- Thread-safe by design
- Safe to cache and share
- Predictable behavior

### Data Flow

```
User Request
    ↓
ChartBuilder (API layer) - Fluent interface for configuration
    ↓
Engines (calculation layer)
    ├── EphemerisEngine → CelestialPosition[]  (planetary positions)
    ├── HouseSystemEngine → HouseCusps          (house systems)
    ├── AspectEngine → Aspect[]                 (aspects)
    └── OrbEngine → orb calculations
    ↓
Components (optional calculations)
    ├── ArabicPartsCalculator → Arabic Parts
    ├── MidpointCalculator → Midpoints
    ├── DignityComponent → Essential/Accidental Dignities
    └── PatternAnalysisEngine → Aspect Patterns
    ↓
CalculatedChart (immutable result)
    ↓
Presentation/Visualization/Export
    ├── ReportBuilder → Terminal reports (Rich)
    ├── ChartRenderer → SVG charts
    └── to_dict() → JSON export
```

---

## Directory Structure

```
/home/user/stellium/
├── src/stellium/              # Main package source code
│   ├── __init__.py            # Public API exports
│   ├── core/                  # Core abstractions (NEVER import from engines here)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [katelouie/stellium](https://github.com/katelouie/stellium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
