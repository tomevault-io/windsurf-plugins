---
trigger: always_on
description: > Computational toolkit for enzyme-polymer conjugate MD simulations.
---

# PolyzyMD — Agent Instructions

> Computational toolkit for enzyme-polymer conjugate MD simulations.
> Python >=3.10 | MIT License | hatchling build | src layout

## Environment

**All simulation-stack commands MUST use a PolyzyMD pixi environment:**

```bash
pixi run -e <env> <command>
```

The PolyzyMD pixi environments contain OpenMM, OpenFF, MDAnalysis and other
heavy dependencies resolved from conda-forge. Never `pip install` these
outside the managed pixi environment.

**Quick commands:**

| Task | Command |
|------|---------|
| Install env | `pixi install -e build` |
| Activate shell | `pixi shell -e build` |
| Run tests | `pixi run -e build pytest tests/ -v` |
| Lint | `ruff check src/` |
| Format | `black src/ --check` (or `black src/` to fix) |
| Build docs | `pixi run -e build make -C docs clean html` |
| Type check | `pixi run -e build mypy src/polyzymd` |

## Git Workflow

- **Branches:** `main` (stable), `dev` (integration), `feature/*` (work)
- Currently on `feature/analysis-module` (25 commits ahead of main)
- Commit messages: imperative mood, 50-char subject, reference issues (`#20`)
- Run `ruff check` and `black --check` before committing
- Never force-push to `main` or `dev`

## Architecture Quick Reference

```
src/polyzymd/
├── cli/          # Click CLI (main.py = entry point)
├── config/       # Pydantic v2 models (schema.py), YAML loading
├── builders/     # System construction (PDB → parameterized topology)
├── simulation/   # OpenMM simulation runners
├── workflow/     # Orchestration (build → simulate → analyze)
├── core/         # Base classes, shared types
├── analysis/     # Post-simulation analysis (contacts, RMSD, etc.)
├── compare/      # Multi-condition comparison engine
├── exporters/    # GROMACS/other format exporters
├── data/         # Bundled data files (force fields, templates)
├── utils/        # Shared utilities
└── configs/      # Default YAML configs
```

## Key Patterns

- **Chain convention:** A=protein, B=substrate, C=polymer, D+=solvent
- **Factory pattern:** `ClassName.from_config(config)` or `ClassName.from_yaml(path)`
- **Lazy imports:** Heavy deps (OpenMM, MDAnalysis) imported inside functions/methods
- **ABC + Strategy:** `ContactCriteria`, `MolecularSelector`, `MoleculeCharger`
- **Registry pattern:** `ComparatorRegistry`, `PlotterRegistry` for extensibility
- **Config:** Pydantic v2 `BaseModel` subclasses with `model_validator`

### Contributor Entry Points for Analysis

When adding a new analysis type or result class, start with these two base
class docstrings — they contain step-by-step instructions:

| Base Class | Location | What It Documents |
|------------|----------|-------------------|
| `BaseAnalyzer` | `analysis/core/registry.py` | How to add a new analyzer (compute, aggregate, register, CLI) |
| `BaseAnalysisResult` | `analysis/results/base.py` | Serialization contract (save/load, field conventions, migration) |

Key rules from those docstrings:

- **Results**: Inherit `BaseAnalysisResult`, set `analysis_type` as `ClassVar[str]`,
  implement `summary()`. Do NOT reimplement `save()`/`load()`.
- **Analyzers**: Implement `analysis_type()`, `from_config()`, `compute()`,
  `compute_aggregated()`, and a `label` property. Register settings with
  `@AnalysisSettingsRegistry.register()`.
- **Nested data objects** (e.g., per-residue stats) inherit `BaseModel`, not
  `BaseAnalysisResult`.
- **Large binary data** (e.g., per-frame SASA) uses NPZ + JSON sidecar instead
  of `BaseAnalysisResult`.

## Design Principles (Critical for Contributors)

This project prioritizes **extensibility** so users can contribute new analyses,
comparators, and plotters without modifying core code. Follow these principles:

### Open-Closed Principle (OCP)

Classes should be **open for extension, closed for modification**. Use:
- Abstract base classes (`ABC`) with well-defined contracts
- Registry patterns for runtime discovery of new implementations
- Strategy pattern for swappable algorithms

**Example:** To add a new plotter, inherit from `BasePlotter` and register
with `@PlotterRegistry.register()`. No changes to `ComparisonPlotter` needed.

### Follow Established Contracts

When extending a registry-based system, **study existing implementations first**:

1. **Read the base class docstrings** — they define the contract
2. **Study 2-3 existing implementations** — understand the expected data flow
3. **Match the pattern exactly** — don't invent new data passing mechanisms

**Anti-pattern to avoid:**
```python
# WRONG: Expecting custom kwargs that the orchestrator doesn't provide
def plot(self, data, labels, output_dir, **kwargs):
    result = kwargs.get("comparison_result")  # Orchestrator never passes this!
    ...
```

**Correct pattern:**
```python
# RIGHT: Load data from filesystem paths provided in data dict
def plot(self, data, labels, output_dir, **kwargs):
    for label in labels:
        analysis_dir = data[label]["analysis_dir"]
        result = MyResult.load(analysis_dir / "my_result.json")
    ...
```

### Registry Pattern Contracts

| Registry | Base Class | Data Source | Key Contract |
|----------|------------|-------------|--------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joelaforet/polyzymd](https://github.com/joelaforet/polyzymd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
