---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**GeistFabrik** is a Python-based divergence engine for Obsidian vaults that generates creative suggestions through both code and Tracery grammars. The name comes from German for "spirit factory" - the system is called GeistFabrik, and individual generative prompts are called "geists."

Inspired by Gordon Brander's work on tools for thought, it implements "muses, not oracles" - suggestions that are provocative rather than prescriptive, generating "What if...?" questions rather than answers.

## Current Project State

**Version**: 0.10.0 (Beta)
**Status**: Feature-complete, release-candidate quality
**Tests**: All passing ✅ (100%)
**Code**: ~21,000 lines across 99 Python files under `src/geistfabrik`

This repository contains:
- **src/geistfabrik/**: Complete implementation of all core modules
  - **default_geists/**: 70 bundled geists (58 code, 12 Tracery) - automatically available
    - _Counts programmatically verified via src/geistfabrik/default_geists/__init__.py_
- **tests/**: Comprehensive test suite (all passing)
- **examples/**: Learning materials demonstrating extension patterns (NOT for installation)
- **specs/**: Original technical specifications (all implemented)
- **testdata/**: Sample Obsidian vault notes from kepano's vault for testing
- **models/**: Bundled sentence-transformers model (all-MiniLM-L6-v2) in Git LFS

The system is fully functional and operational. All phases of the specification have been implemented.

### Default Geists vs Examples

**Important distinction:**
- **Default geists** (src/geistfabrik/default_geists/): 70 bundled geists that work automatically
  - Users can enable/disable via config.yaml
  - No installation needed - they're part of the package
- **Examples** (examples/): Learning materials showing extension patterns
  - NOT meant to be installed into vaults
  - Demonstrate how to write custom geists, functions, metadata modules
  - For reference and learning only

## Development Workflow (CRITICAL)

**BEFORE COMMITTING OR PUSHING CODE, ALWAYS FOLLOW THIS WORKFLOW:**

### Pre-Commit (Automatic)
Pre-commit hooks run automatically on `git commit`:
- Ruff linting and formatting
- Trailing whitespace removal
- YAML validation
- Basic checks

### Before Pushing (MANDATORY)

**Run the validation script:**
```bash
./scripts/validate.sh
```

This script runs the **same checks as CI** (same tools, same marker filter):
1. `ruff check src/ tests/` - Linting
2. `mypy src/ --strict` - Type checking (STRICT MODE)
3. `python scripts/detect_unused_tables.py` - Database validation
4. `bandit -c pyproject.toml -r src/geistfabrik -ll -q` - Security scan
5. `pytest tests/unit -v -m "not slow and not benchmark"` - Unit tests (with mocked models)
6. `pytest tests/integration -v -m "not slow and not benchmark"` - Integration tests (with mocked models)
7. `python scripts/check_phase_completion.py` - Acceptance-criteria gate: *runs*
   every machine-verifiable criterion in `specs/acceptance_criteria.md` (it does
   not trust the status column) so the spec cannot silently drift from the code.
   Each criterion is AUTO (a real command is executed) or MANUAL (prose,
   reported but non-gating). See that file's header for the contract.

**If validate.sh passes, CI will almost certainly pass. If it fails, DO NOT PUSH.**

Caveat: validate.sh runs in your single local environment. CI *additionally*
exercises Python 3.11 **and** 3.12, macOS, and the `[vector-search]` extra
(sqlite-vec). A failure that is specific to those (a 3.12-only typing quirk, a
macOS path issue, or the sqlite-vec backend) can still pass locally - so a
green validate.sh is necessary but, for those environment-specific cases, not
absolutely sufficient.

**Note on model downloads**: The validation script uses mocked sentence-transformers models (via `SentenceTransformerStub` in `tests/conftest.py`) so it works in environments without Git LFS or network access (like Claude Code for Web). The `-m "not slow"` flag triggers automatic model mocking. Only 9 tests marked as "slow" require the real model (~90MB).

### Common Mistakes to Avoid

❌ **NEVER** run custom variations of CI checks:
```bash
# These are WRONG and will cause CI failures
mypy src/geistfabrik --ignore-missing-imports
mypy src/ --config-file mypy.ini
pytest tests/ -k "unit"
```

✅ **ALWAYS** use the validated script:
```bash
./scripts/validate.sh
```

### Type Checking Requirements

CI uses `mypy --strict` which requires:
- Explicit type parameters for generics: `dict[str, Any]` not `dict`
- Type hints on all function parameters and returns
- No implicit `Any` types

**Type Hint Style**: GeistFabrik uses **modern Python 3.9+ syntax** (PEP 585) for built-in types:
- Use `list[Type]`, `dict[K, V]`, `tuple[T, ...]` (lowercase, no imports)
- NOT `List[Type]`, `Dict[K, V]`, `Tuple[T, ...]` (from `typing`)

**Example - Missing Type Parameters** (This will fail CI):
```python
# ❌ WRONG - Missing type parameters
def from_dict(cls, data: dict) -> Config:  # dict needs [str, Any]
    pass

# ✅ CORRECT - Type parameters provided
from typing import Any


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adewale/geist_fabrik](https://github.com/adewale/geist_fabrik) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
