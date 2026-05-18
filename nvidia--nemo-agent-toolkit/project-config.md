---
trigger: always_on
description: Follow these rules when creating, modifying, or generating any code, tests, documentation, or configuration files
---


# NeMo Agent Toolkit General Coding Guidelines

These are the overarching standards that every **source, test, documentation and CI file** in this repository must follow. Adhering to these rules locally ensures the project's automated checks and pipelines succeed on your first push.

---

## Terminology and Naming

- Make sure to follow this naming convention for all the documentation. If there is any documentation not following this rule, you MUST update it.
- **Full name (first use)**: "NVIDIA NeMo Agent Toolkit" — use for document titles, webpage headers, and any public descriptions
- **Short name (subsequent references)**: "NeMo Agent Toolkit" or "the toolkit"
- **Capitalization rules**:
  - In document titles, headings, or any context where all words are capitalized, use "Toolkit" (capital T): e.g., "NVIDIA NeMo Agent Toolkit" or "NeMo Agent Toolkit"
  - In all other contexts (body text, descriptions), use "toolkit" (lowercase t): e.g., "NVIDIA NeMo Agent Toolkit" or "NeMo Agent Toolkit"
- **Technical identifiers** (code, CLI, packages, URLs):
  - `nat` for the API namespace and CLI tool
  - `nvidia-nat` for the package name
  - `NAT_` prefix for environment variables
  - `NeMo-Agent-Toolkit` for URLs, directory names, and contexts where capitalization is preserved (no underscores or spaces)
- **"NAT" abbreviation**:
  - OK in code comments
  - NEVER use "NAT" or "nat" to refer to the toolkit in documentation
- Examples:
  - "In the NeMo Agent Toolkit, you can…"
  - "Change directory to the NeMo Agent Toolkit repo root…"
  - Heading: "# Getting Started with NeMo Agent Toolkit"
- Consistently use this terminology throughout all documentation
- NeMo Agent Toolkit was previously known as the Agent Intelligence toolkit, and AgentIQ. You should NEVER use the deprecated names, including Agent Intelligence toolkit, aiqtoolkit, AgentIQ, or AIQ/aiq. If you see any of these names in the documentation, you should update it based on the latest naming convention above, unless those names are intentionally used to refer to the deprecated names, or implementing a compatibility layer for the deprecated names.
- DO NOT change the content of `CHANGELOG.md`
- AIQ Blueprint is the intended name for the blueprint. DO NOT change it.

## Project Structure

- All importable Python code lives under `src/` or `packages/<pkg>/src/` so namespace-packages resolve correctly.
- Each example is an installable package in `examples/<example_name>` and exposes an `__main__.py` for `python -m <example_name>` execution.
- **Examples directory guidelines**:
  - At minimum, examples should contain a `README.md` or `README.ipynb` file.
  - Python code should be placed in a `src/` subdirectory with a `pyproject.toml` file.
  - Scripts should be placed in a `scripts/` subdirectory (optional).
  - YAML files should be placed in a `configs/` subdirectory.
  - Sample data files should be placed in a `data/` subdirectory and checked into git-lfs.
- **Packages directory guidelines**:
  - Each package should contain a `pyproject.toml` file.
  - The `pyproject.toml` should declare a dependency on `nvidia-nat` or another package with a name starting with `nvidia-nat-`.
  - Dependencies should use `~=<version>` format with two-digit versions (e.g., `~=1.0`).
  - If packages contain Python code, they should have tests in a `tests/` directory at the same level as `pyproject.toml`.
- Unit tests live in `tests/` (or `examples/*/tests`) and use the markers defined in `pyproject.toml` (e.g. `integration`).
- Documentation sources are Markdown files under `docs/source`. Image files should be placed in `docs/source/_static` directory.
- Configuration files consumed by code are stored next to that code in a `configs/` folder.
- Large / binary assets **must** be committed with Git-LFS and placed in a neighbouring `data/` folder.
- Shell or utility scripts belong in `scripts/` or `ci/scripts/` – never mix them with library code.
- **Source code organization**:
  - `packages/nvidia_nat_core/src/nat/**/*`: Contains core functionality. Changes should prioritize backward compatibility.

## Code Formatting & Imports

- Follow [PEP 20](https://peps.python.org/pep-0020/) and [PEP 8](https://peps.python.org/pep-0008/) for Python style guidelines.
- Run **yapf** second (PEP 8 base, `column_limit = 120`).
- Indent with 4 spaces, never tabs, and ensure every file ends with a single newline.
- CI fails if formatting is wrong; run `pre-commit run --all-files` locally before pushing.

## Linting

- **ruff** (via `ruff check --fix`) also runs via pre-commit; is executed using the configuration embedded in `pyproject.toml`,
  fix warnings unless they're explicitly ignored in `pyproject.toml`. ruff is only used as a linter not for formatting.
- Respect the naming schemes: `snake_case` for functions & variables, `PascalCase` for classes, `UPPER_CASE` for constants.

## Type Hints

- All public APIs require Python 3.11+ type hints on parameters and return values.
- Prefer `collections.abc` / `typing` abstractions (`Sequence` over `list`).
- Use `typing.Annotated` for units or extra metadata when useful.
- Treat `pyright` warnings (configured in `pyproject.toml`) as errors during development.

## Exception Handling


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/NeMo-Agent-Toolkit](https://github.com/NVIDIA/NeMo-Agent-Toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
