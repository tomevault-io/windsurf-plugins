---
trigger: always_on
description: Metatrain is a **CLI for training & evaluating machine learning models for atomistic systems**. It provides a unified YAML-based interface (`mtt train options.yaml`) for multiple ML architectures (SOAP-BPNN, PET, GAP, MACE, FlashMD, LLPR) with automatic TorchScript export compatibility for MD engines (LAMMPS, ASE, i-PI). New architectures follow a standardized plugin system pattern where each architecture is discoverable via `metatrain.utils.architectures.import_architecture()`.
---

# Copilot Instructions

Metatrain is a **CLI for training & evaluating machine learning models for atomistic systems**. It provides a unified YAML-based interface (`mtt train options.yaml`) for multiple ML architectures (SOAP-BPNN, PET, GAP, MACE, FlashMD, LLPR) with automatic TorchScript export compatibility for MD engines (LAMMPS, ASE, i-PI). New architectures follow a standardized plugin system pattern where each architecture is discoverable via `metatrain.utils.architectures.import_architecture()`.

## Repository Structure

| Path                                     | Description                                              |
| ---------------------------------------- | -------------------------------------------------------- |
| `.github/`                               | GitHub workflows and issue templates                     |
| `developer/`                             | Helper scripts and tools for development                 |
| `docs/`                                  | Sphinx documentation in reStructuredText                 |
| `examples/`                              | Training examples and YAML config templates rendered     |
|                                          | with sphinx-gallery for the documentation                |
| `src/metatrain/`                         | Python source code                                       |
| `src/metatrain/cli/`                     | Command-line interface (train, eval, export)             |
| `src/metatrain/deprecated/`              | Deprecated architectures (currently empty)               |
| `src/metatrain/experimental/`            | Experimental architectures (e.g., mace, flashmd)         |
| `src/metatrain/{soap_bpnn,pet,gap,...}/` | Stable architecture implementations (plugin system)      |
| `src/metatrain/utils/`                   | Shared utilities (data, I/O, losses, testing)            |
| `tests/`                                 | Test suite using pytest                                  |
| `CITATION.cff`                           | Citation metadata for the project                        |
| `CODEOWNERS`                             | GitHub code owners for review assignment                 |
| `CONTRIBUTING.rst`                       | Contributing guidelines in reStructuredText              |
| `LICENSE`                                | BSD-3-Clause license                                     |
| `MANIFEST.in`                            | Files to include in package distribution                 |
| `README.md`                              | Project overview and quick start                         |
| `pyproject.toml`                         | Python project configuration and dependencies            |
| `tox.ini`                                | Testing and development environment configuration        |

## Environment Setup

Install dependencies and setup for development:

```bash
pip install tox
git clone https://github.com/metatensor/metatrain
cd metatrain
pip install -e .
```

To see all available tox environments:

```bash
tox list
```

## Testing

All tests are executed via `tox`. Common commands:

```bash
tox                  # Run all tests (lint, format, tests, architecture tests)
tox -e tests         # Core library unit tests only
tox -e lint          # Code style, type hints, docstrings
tox -e format        # Auto-format code (ruff, yamlfix, prettier)
tox -e {arch}-tests  # Architecture-specific tests (e.g., tox -e soap-bpnn-tests)
tox -e docs          # Build documentation
tox -e tests -- tests/utils/test_abc.py  # Run specific test file
tox -e tests -- tests/test_init.py::test_some_function  # Run specific test function
```

Tests use `pytest` framework with fixtures in `tests/resources/`. Architecture tests inherit from `metatrain.utils.testing.ArchitectureTests` which provides reusable test classes (Input, Output, Training, Checkpoints, TorchScript, Exported).

**Development Tip**:

- Test setup (generating reference outputs via `bash {toxinidir}/tests/resources/generate-outputs.sh`) can take a while. For faster iteration during development, you can temporarily comment out this step in `tox.ini` under `[testenv:tests]` → `commands_pre`, then remember to uncomment before final testing.
- tox creates a private .tox/ folder for virtual environments. To avoid issues with cached dependencies, you can delete this folder and re-run `tox` to recreate clean environments.

## Training & Export Pipeline

The core workflow follows this sequence:

1. Parse YAML config with `OmegaConf` and validate via `utils/pydantic.py`
2. Dynamic architecture loading: `import_architecture(name)` returns module with `__model__` and `__trainer__`
3. Load datasets: `metatrain.utils.data.get_dataset()` reads XYZ files via `ase.io.read()`
4. Training: `Trainer.train()` runs forward passes, loss computed via `utils/loss.py`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [metatensor/metatrain](https://github.com/metatensor/metatrain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
