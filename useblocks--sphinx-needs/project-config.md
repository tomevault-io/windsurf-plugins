---
trigger: always_on
description: The delta for this package. Everything repository-level — the workspace layout, the
---

# AGENTS.md — packages/sphinx-codelinks

The delta for this package. Everything repository-level — the workspace layout, the
commands, the lock, lint/format/type-check configuration, the release recipe, the pull
request requirements and the commit-message convention — is in the ROOT
[`AGENTS.md`](../../AGENTS.md), and this file does not repeat it. What is here is what an
agent has to know that is true of sphinx-codelinks and not of the workspace.

## Project Overview

sphinx-codelinks is a Sphinx extension that provides fast source code traceability for
sphinx-needs. It:

- **analyses source code** — scans C, C++, C#, Python, Rust, Go, YAML, JSON and Bash files
  for special comment markers, with tree-sitter;
- **creates needs from them** — turns discovered markers into sphinx-needs items;
- **traces sources** — links documentation to exact source lines, and generates a
  syntax-highlighted HTML page per traced file with line anchors;
- **has a CLI** — `codelinks analyse`, `codelinks discover` and `codelinks write rst`,
  for use outside a Sphinx build.

It is the only member of this workspace whose `src/` imports `sphinx_needs`, and it
declares it as a **runtime** dependency with a tight floor (`sphinx-needs>=8.5.0,<9`,
which `check_workspace.py` check (4) enforces against the sibling's current version, and
`propagate_floors.py` moves at each sphinx-needs release).

## Package structure

```text
pyproject.toml          # `[project]`, `[project.urls]`, `[build-system]` and nothing else:
                        #   ruff, ty, pytest and the dependency groups are the ROOT's
.readthedocs.yaml       # this package's RTD project; every path in it is relative to the
                        #   REPOSITORY root, not to the file
README.md · LICENSE
design/                 # import-commit-map.txt: old hash -> new hash for the 2026-09 import

src/sphinx_codelinks/   # Main source code
├── __init__.py         # `__version__` (public, in `__all__`) and the Sphinx `setup()`
├── cmd.py              # CLI commands using Typer
├── config.py           # Configuration dataclasses + TypedDicts, and the TOML loader
├── logger.py           # Logging utilities
├── needextend_write.py # Write RST files with Sphinx-Needs directives
├── analyse/            # Code analysis module
│   ├── analyse.py      # Main analysis orchestration
│   ├── models.py       # dataclasses/TypedDicts/Enums for analysis results
│   ├── oneline_parser.py # One-line comment parser
│   ├── projects.py     # Project-specific analyzers (C++, Python, etc.)
│   ├── utils.py        # Analysis utilities, including the git-root helpers
│   └── preproc/        # the OPTIONAL libclang engine -- see below
├── source_discover/    # Source file discovery
│   ├── config.py       # Discovery configuration
│   └── source_discover.py # File discovery logic
└── sphinx_extension/   # Sphinx extension components
    ├── source_tracing.py # Main Sphinx extension setup
    ├── html_wrapper.py  # HTML output wrapper for traced source
    ├── debug.py         # Debug utilities
    ├── ub_sct.css       # CSS for source tracing UI
    └── directives/      # Custom Sphinx directives

tests/                  # Test suite -- `tests/__init__.py` is why this path is NOT in the
├── __init__.py         #   root `testpaths` (see the root AGENTS.md)
├── conftest.py         # Pytest fixtures and configuration
├── test_*.py           # 16 test modules
├── __snapshots__/      # Syrupy snapshot test fixtures
├── data/               # Test data and fixtures
└── doc_test/           # minimal Sphinx projects for the integration tests

docs/                   # Documentation source (RST) -- conf.py sits IN the source dir,
├── conf.py             #   so `sphinx-build docs docs/_build/html` needs no `-c`
├── ubproject.toml      # this docs project's own needs + codelinks configuration
├── changelog.rst       # `bump.py` stamps this path; do not move it
├── index.rst · basics/ · components/ · development/ · _static/
```

## The two facts that are workspace-specific

### libclang is optional, and 56 tests depend on it

The preprocessor-aware C/C++ engine (`analyse/preproc/`) needs `clang.cindex`, which comes
from the `libclang` wheel. It is optional at runtime — the member's `libclang` extra — and
`analyse/preproc/__init__.py` imports the loader **eagerly**, so importing anything under
that package without the wheel raises.

In this workspace the wheel is the root dependency group **`codelinks-libclang`**, not part
of `test`: it is 23 MiB and 81 MB on disk, and every cell of every package would otherwise
pay for it. Every `test-codelinks*` poe task adds the group, and so does CI's Extensions
cell.

**`test-codelinks` syncs the group into the DEFAULT `.venv`.** It has no
`UV_PROJECT_ENVIRONMENT` of its own, unlike its three `-sphinx7/8/9` siblings, so the wheel
lands in the environment every other command uses — and the next plain `uv sync --frozen`
prunes it out again (`Uninstalled 1 package: - libclang==18.1.1`). So the two numbers only
appear either side of that sync, and this is the sequence that shows both:

```bash
uv run poe test-codelinks                                            # 359 passed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [useblocks/sphinx-needs](https://github.com/useblocks/sphinx-needs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
