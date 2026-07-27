---
trigger: always_on
description: **Drudge** is a symbolic algebra system for tensorial and noncommutative algebras, built on top of SymPy. It specializes in quantum chemistry and many-body theory applications, focusing on symbolic manipulation of indexed quantities, summations, and noncommutative algebraic systems.
---

# Drudge Repository - Copilot Instructions

## Repository Overview

**Drudge** is a symbolic algebra system for tensorial and noncommutative algebras, built on top of SymPy. It specializes in quantum chemistry and many-body theory applications, focusing on symbolic manipulation of indexed quantities, summations, and noncommutative algebraic systems.

### Key Features
- Symbolic tensor algebra with symmetry support
- Canonical forms for tensor expressions with permutational symmetries
- Noncommutative algebras (CCR/CAR, Clifford algebras, su(2), and more)
- C++ extensions for performance-critical Wick contractions and canonicalization operations
- Integration with companion package [gristmill](https://github.com/DrudgeCAS/gristmill) for code generation

### Repository Statistics
- **Languages**: Python (primary), C++ (extensions), DRS (Drudge scripts, a domain-specific language based on Python)
- **Size**: ~16K lines of Python code, >160 test cases
- **Python Version**: 3.12+ (currently testing on 3.13)
- **C++ Standard**: C++14
- **Dependencies**: SymPy, PySpark/DummyRDD, Jinja2, C++ compiler

## Build and Development Instructions

### Environment Setup (ALWAYS follow this order)

1. **Always initialize submodules first:**
   ```bash
   git submodule update --init --recursive
   ```

2. **Always install uv before building** (if not available):
   ```bash
   # uv is the required package manager
   pip install uv
   ```

3. **Install dependencies and build:**
   ```bash
   uv sync --locked --extra dev
   ```

4. **CRITICAL: Always set DUMMY_SPARK=1 for testing:**
   ```bash
   export DUMMY_SPARK=1
   ```
   This environment variable is **required** for all test runs. Without it, tests may fail or behave unexpectedly as the pyspark version (v2.4) used in this package is outdated (The current pyspark version is v4.0). We are in the process of migrating from pyspark to dask.

### Build Commands

| Command | Purpose | Time | Notes |
|---------|---------|------|-------|
| `uv sync --locked --extra dev` | Install all dependencies | ~7s | Always run after clone |
| `uv build` | Build package distribution | ~30s | Creates wheel and source dist |
| `uv run pytest tests/` | Run full test suite | ~24s | Requires DUMMY_SPARK=1 |

### Testing and Validation

**Test Environment Setup:**
```bash
# ALWAYS set this before running tests
export DUMMY_SPARK=1
uv run pytest tests/ -v
```

**Test Validation Steps:**
1. All tests should pass with only warnings about pdflatex (safe to ignore)
2. Test runtime should be under 30 seconds
3. No errors about missing SparkContext when DUMMY_SPARK=1 is set

### Common Issues and Workarounds

**Build Issues:**
- **Missing submodules**: Always run `git submodule update --init --recursive` after cloning the repo
- **C++ compilation warnings**: Some warnings about uninitialized variables are expected and safe
- **License deprecation warning**: Expected warning about pyproject.toml license format

**Test Issues:**
- **PySpark errors**: Ensure `export DUMMY_SPARK=1` is set before running tests
- **Missing pdflatex warning**: Safe to ignore; only affects LaTeX report generation

## Project Layout and Architecture

### Root Directory Structure
```
drudge/
├── .github/workflows/       # CI/CD pipelines
│   ├── ci.yml               # Main CI (Ubuntu/macOS)
│   ├── copilot-setup-steps.yml  # Setup reference  
│   └── windows.yml          # Windows builds (experimental)
├── deps/libcanon/           # Git submodule for C++ canonicalization
├── docs/                    # Sphinx documentation
│   ├── examples/            # Example scripts and notebooks
│   ├── conf.py              # Sphinx configuration
│   └── Makefile             # Documentation build
├── drudge/                  # Main Python package
├── tests/                   # Test suite (pytest)
├── CMakeLists.txt           # CMake configuration for C++ extensions
├── pyproject.toml           # Modern Python project config
├── MANIFEST.in              # Package data inclusion
└── uv.lock                  # Locked dependencies
```

### Main Package Structure (`drudge/`)
- `__init__.py` - Package exports and version (v0.11.0)
- `__main__.py` - Entry point for running drudge as module (`python -m drudge`)
- `_tceparser.py` - Tensor Contraction Engine (TCE) output parser for TCE integration
- `bcs.py` - Reduced BCS (pairing) Hamiltonian drudge implementation
- `canon.py` - Canonicalization algorithms for tensor expressions
- `canonpy.cpp/.h` - C++ extension for permutation groups and canonicalization
- `clifford.py` - Clifford algebra implementation
- `drs.py` - Drudge script system (domain-specific language)
- `drudge.py` - Core Drudge class and tensor data structure
- `fock.py` - Fermionic/bosonic operator algebra on Fock spaces
- `genquad.py` - General quadratic algebra implementation
- `nuclear.py` - Nuclear physics utilities
- `report.py` - HTML/LaTeX report generation for symbolic results
- `su2.py` - su(2) algebra implementation
- `templates/` - Jinja2 templates for generating HTML/LaTeX reports

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DrudgeCAS/drudge](https://github.com/DrudgeCAS/drudge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
