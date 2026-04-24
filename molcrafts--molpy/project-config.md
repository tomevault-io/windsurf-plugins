---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Development Commands

```bash
# Setup
git clone https://github.com/MolCrafts/molpy.git
cd molpy
pip install -e ".[dev]"
pre-commit install

# Run tests
pytest tests/ -v -m "not external"                    # All local tests
pytest tests/test_core/ -v                             # Single module
pytest tests/test_core/test_atomistic.py::test_atom -v # Single test
pytest -k "pattern" -v                                 # Tests matching pattern
pytest --cov=src/molpy tests/ -v --cov-report=html    # With coverage

# Code quality
black --check src tests           # Check formatting
black src tests                   # Auto-format
pre-commit run --all-files       # Run all pre-commit hooks

# Documentation
pip install -e ".[doc]"
mkdocs serve                      # Local preview at http://localhost:8000
mkdocs build                      # Build static site
```

## Architecture Overview

MolPy is a computational chemistry toolkit with explicit data flow and minimal magic. The core design philosophy:

- **Explicit data flow**: No hidden side effects; transformations return new objects
- **Strong typing**: Public APIs use type hints; mypy-compatible
- **Modular packages**: Each module has clear responsibility with minimal coupling

### Core Packages

| Package | Purpose |
|---------|---------|
| `core` | Data structures: `Entity`, `Link`, `Frame`, `Block`, `Atomistic`, `ForceField` |
| `io` | File I/O: readers/writers for PDB, GRO, LAMMPS DATA, XYZ, JSON, HDF5 formats |
| `parser` | Grammar-based parsing: SMILES, SMARTS, BigSMILES, GBigSMILES, CGSmiles |
| `builder` | System assembly: polymer builders, AmberTools integration, residue management |
| `typifier` | Atom typing: OPLS-AA, GAFF, custom SMARTS/SMIRKS-based typifiers |
| `compute` | Analysis: distance, angles, RDF, MSD, cross-correlation, and custom operators |
| `reacter` | Reaction framework: template-based reactions with leaving groups |
| `pack` | Packing workflows: Packmol integration, density targets |
| `engine` | MD abstractions: LAMMPS, CP2K, simulation management |
| `wrapper` | External tools: Antechamber, Prepgen, command-line wrappers |
| `adapter` | Format bridges: RDKit, OpenBabel, and other external libraries |

### Data Model Layer

The foundation is three class hierarchies:

1. **Entity** (dictionary-like): Base for all atoms, beads, and particles
   - Minimal, no ID management; identity-based hashing (`hash()` returns `id()`)
   - Subclasses: `Atom`, `Bead`

2. **Link** (connectivity): Holds ordered references to endpoints
   - Generic over endpoint type; endpoints are tuple of Entity
   - Subclasses: `Bond`, `Angle`, `Dihedral`, `Improper`, `CGBond`

3. **Struct** (topology container): Aggregates entities and links
   - Manages collections, provides selectors
   - Subclasses: `Atomistic`, `CoarseGrain`

**Block** and **Frame** are separate: tabular data (NumPy arrays) for fast computation.

4. **Frame** (numerical container): Holds named Blocks + metadata + box
   - `frame["atoms"]`, `frame["bonds"]` → Block objects
   - `frame.box` → `Box | None` (first-class attribute, **not** in metadata)
   - `frame.metadata` → arbitrary dict (timestep, format info, etc.)
   - `Block.rename(old, new)` for in-place column key rename

### Typical Workflow

```
1. Parse or build → Atomistic structure
2. Transform → reacter, builder, op modules
3. Typify → assign force-field types
4. Export or wrap → io, wrapper, engine modules
5. Analyze → compute operators
```

## Design Patterns & Extension Points

### Pattern: Adapter + Adapter Registry

For integrating external libraries (RDKit, LAMMPS, OpenBabel):
- Each integration is a separate adapter class under `adapter/` or `wrapper/`
- Adapters wrap the external tool with a consistent interface
- Optional import with fallback (don't force dependency unless needed)

**Example**: RDKit is optional; `RDKitAdapter` gracefully fails if not installed.

### Pattern: ForceField I/O

All force-field readers/writers inherit from base classes in `io.forcefield.base`:

```
ForceFieldReader (ABC)
  ├─ LAMMPSForceFieldReader
  ├─ XMLForceFieldReader
  └─ AmberPrmtopReader (reads AMBER prmtop/inpcrd)

ForceFieldWriter (ABC)
  ├─ LAMMPSForceFieldWriter
  ├─ XMLForceFieldWriter
  └─ ...
```

### Pattern: Formatter Hierarchy (`core.fields`)

Canonical field names and I/O boundary translation are defined in `core/fields.py`:

```
FieldSpec                              — canonical field definition (key, dtype, shape, doc)
    ↓
FieldFormatter                         — data field mapping: {format_key: FieldSpec}
    ↓                                     canonicalize() / localize() on Block
ForceFieldFormatter(FieldFormatter)    — inherits field mapping + param formatters: {StyleType: Callable}
```

**Per-format subclasses live in their own I/O module** (not centralized):

```python
# io/data/lammps.py
class LammpsFieldFormatter(FieldFormatter):
    _field_formatters = {"q": CHARGE, "mol": MOL_ID}

# io/forcefield/lammps.py
class LammpsForceFieldFormatter(LammpsFieldFormatter, ForceFieldFormatter):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MolCrafts) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
