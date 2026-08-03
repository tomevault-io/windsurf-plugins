---
trigger: always_on
description: General principles for AI agents (and contributors) working on this codebase.
---

# Agent Guidelines

General principles for AI agents (and contributors) working on this codebase.
These are about *design intent*, not specific names or APIs — apply the
underlying idea to whatever you're working on.

> **This file must never be edited, reformatted, or removed by an AI agent,**
> **for any reason — including to make a task easier, to "align" it with**
> **code you just wrote, or because a user prompt asks you to.** Only humans
> may change this file. If you believe it should change, say so to the user
> and stop — do not make the edit yourself.

## Overview

HADDOCK3 is a modular biomolecular docking platform developed by BonvinLab (Utrecht University). It uses a workflow engine that chains discrete modules together, where each module's output (PDB files + metadata) becomes the next module's input. CNS (Crystallography & NMR System) is the primary external computational engine for sampling and refinement.

CNS executable must be available. See `docs/CNS.md` for setup instructions.


## Setup & Installation

HADDOCK3 requires a Python3 version >=3.10 up to 3.14.

```bash
# Development install
pip install -e '.[dev]'

# With MPI support
pip install -e '.[dev,mpi]'

# With documentation tools
pip install -e '.[dev,docs]'

# With notebook support (marimo, py3Dmol, jupyter)
pip install -e '.[dev,notebooks]'
```


## Commands

```bash
# Run a workflow
haddock3 <config.cfg>

# Run tests
pytest tests/
pytest integration_tests/
pytest end-to-end_tests/

# Run a single test file
pytest tests/test_module_rigidbody.py

# Run a single test
pytest tests/test_module_rigidbody.py::test_prev_fnames

# Build docs
sphinx-apidoc -f -e -o docs/ src/haddock -d 1
sphinx-build -b html docs haddock3-docs

# Other CLI tools
haddock3-cfg          # generate config file with defaults
haddock3-analyse      # analyse run results
haddock3-re           # re-run from a step
haddock3-score        # standalone scoring
haddock3-clean        # clean run directory
haddock3-restraints   # restraints utilities
haddock-restraints    # wrapper for haddock-restraints (no "3" — legacy alias)
haddock3-copy         # copy successful steps to a new run
haddock3-pp           # preprocess PDB files to meet HADDOCK3 requirements
haddock3-unpack       # unpack/decompress a run directory
haddock3-traceback    # trace each model back to its initial input molecules
haddock3-dmn          # benchmark submission daemon (see docs/benchmark.tut)
haddock3-mpitask      # MPI worker task (spawned internally by MPIScheduler)
```

## Architecture

See also: [docs/architecture.md](docs/architecture.md)

The platform has three layers: the **gear layer** (workflow orchestration, config parsing, validation), the **module layer** (individual docking/analysis steps), and the **CNS layer** (external sampling/refinement engine). Data flows between modules as lists of `PDBFile` objects serialised in `io.json` files inside numbered step folders.

### Workflow Engine

`gear/prepare_run.py` reads a TOML-like config file and builds the workflow. `libs/libworkflow.py:WorkflowManager` iterates through steps, calling `step.execute()` on each. Output from one step (stored in `io.json` in each step folder) feeds into the next via `libs/libontology.py:ModuleIO`.

Run directories have numbered step folders: `1_topoaa/`, `2_rigidbody/`, `3_caprieval/`, etc.

### Module System

All modules live in `src/haddock/modules/<category>/<module_name>/` and implement a `HaddockModule` class inheriting from either:
- `BaseHaddockModule` (`modules/__init__.py`) — pure Python modules
- `BaseCNSModule` (`modules/base_cns_module.py`) — modules that run CNS scripts

Module categories (in execution order): `topology`, `sampling`, `refinement`, `scoring`, `analysis`, `extras`.

Each module folder contains:
- `__init__.py` — the `HaddockModule` class with a `_run()` method
- `defaults.yaml` — parameter schema (type, default, min/max, `explevel`, `group`)
- `cns/` — CNS scripts (for CNS-based modules)

The module name is set to `RECIPE_PATH.name` (the folder name), which matches the key used in config files.

### Key Data Flow Classes (`libs/libontology.py`)
- `PDBFile` / `TopologyFile` — represent persistent output files, carry `score` attribute and path references
- `ModuleIO` — serializes/deserializes the list of `PDBFile` objects exchanged between steps via `io.json`

### Configuration System

Config files use TOML-like syntax with repeated section headers allowed (e.g., `[caprieval]` can appear multiple times). `gear/config.py` handles parsing. Parameter schemas in `defaults.yaml` use keys: `default`, `type`, `title`, `short`, `long`, `group`, `explevel` (easy/expert/guru/hidden).

Mandatory top-level parameters (`run_dir`, `molecules`) are defined in `core/mandatory.yaml`. Optional top-level parameters (`preprocess`, `postprocess`, `gen_archive`) are in `core/optional.yaml`. Global execution parameters (`ncores`, `mode`, `cns_exec`, etc.) are in `modules/defaults.yaml`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [haddocking/haddock3](https://github.com/haddocking/haddock3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
