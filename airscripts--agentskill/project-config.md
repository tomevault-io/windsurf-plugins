---
trigger: always_on
description: agentskill is a single-repo Python CLI published from the `agsk` package metadata and exposed as the `agentskill` console command. It analyzes one or more repositories, emits structured analyzer output, and also supports direct `AGENTS.md` generation and in-place update flows. The codebase is organized around packaged runtime modules under `agentskill/`, thin direct wrappers under `scripts/`, reference specs and fixture repositories at the repo root, and a separate `tests/` tree that exercises a
---

# AGENTS.md

## 1. Overview

agentskill is a single-repo Python CLI published from the `agsk` package metadata and exposed as the `agentskill` console command. It analyzes one or more repositories, emits structured analyzer output, and also supports direct `AGENTS.md` generation and in-place update flows. The codebase is organized around packaged runtime modules under `agentskill/`, thin direct wrappers under `scripts/`, reference specs and fixture repositories at the repo root, and a separate `tests/` tree that exercises analyzer internals, generation/update flows, and CLI entrypoints.

## 2. Repository Structure

```text
agentskill/
  agentskill/
    main.py                 # packaged CLI entry point; argument parsing and dispatch only
    commands/               # analyzer implementations
    lib/                    # orchestration, output, reference, generation, and update helpers
    common/                 # shared low-level helpers and registries
  pyproject.toml            # packaging, pytest, ruff, coverage config
  README.md                 # user-facing overview and command reference
  SYSTEM.md                 # synthesis spec for generated AGENTS.md files
  SKILL.md                  # operational workflow for the skill
  AGENTS.md                 # conventions for this repo
  LICENSE                   # MIT license text
  references/
    GOTCHAS.md              # extraction and synthesis failure modes
  examples/
    python/                 # fixture repository used by analyzer tests
    javascript/             # fixture repository for JS/TS detection paths
    mixed/                  # multi-language fixture repository
    ...                     # additional per-language example repos
  scripts/
    analyze.py              # thin direct-execution wrapper to packaged CLI
    scan.py                 # thin direct-execution wrapper
    measure.py              # thin direct-execution wrapper
    config.py               # thin direct-execution wrapper
    git.py                  # thin direct-execution wrapper
    graph.py                # thin direct-execution wrapper
    symbols.py              # thin direct-execution wrapper
    tests.py                # thin direct-execution wrapper
    generate.py             # thin direct-execution wrapper to packaged CLI
    update.py               # thin direct-execution wrapper to packaged CLI
  tests/                    # pytest suite; separate tree, not colocated
    conftest.py             # sys.path test bootstrap
    test_support.py         # shared repo/setup helpers for tests
```

- New analyzer logic goes in `agentskill/commands/`, not in entrypoint wrappers.
- Shared CLI plumbing, generation, reference adaptation, and update flows belong in `agentskill/lib/`; low-level reusable helpers belong in `agentskill/common/`.
- Files under `scripts/*.py` stay as thin wrappers around packaged command entrypoints such as `agentskill.commands.<name>.main` or `agentskill.main`.
- New tests go in `tests/` as `test_<subject>.py`; this repo does not colocate tests beside source files.
- New fixture repos or language-shape examples belong in `examples/`, not mixed into `references/`.
- Specs and extraction notes belong in `README.md`, `SYSTEM.md`, `SKILL.md`, and `references/`, not inside runtime modules.
- Keep the repo root small: metadata, docs/spec files, and no business logic outside `agentskill/`.

## 5. Commands and Workflows

```bash
# Install editable package with dev tooling
python -m pip install -e '.[dev]'

# Optional local hooks
pre-commit install

# Run all analyzers
agentskill analyze <repo> --pretty

# Generate or update markdown
agentskill generate <repo>
agentskill generate <repo> --out AGENTS.md
agentskill generate <repo> --interactive
agentskill update <repo>
agentskill update <repo> --section testing
agentskill update <repo> --force

# Run individual analyzers through the installed CLI
agentskill scan <repo> --pretty
agentskill measure <repo> --lang python --pretty
agentskill config <repo> --pretty
agentskill git <repo> --pretty
agentskill graph <repo> --pretty
agentskill symbols <repo> --pretty
agentskill tests <repo> --pretty

# Direct wrapper execution
python scripts/analyze.py <repo> --pretty
python scripts/scan.py <repo> --pretty
python scripts/generate.py <repo>
python scripts/update.py <repo>

# Local checks
ruff format .
ruff check .
mypy
pytest
```

- `python -m pip install -e '.[dev]'` is the documented development install path; use it instead of reconstructing the dev dependency list manually.
- `agentskill analyze <repo> --pretty` is the canonical aggregate analyzer workflow.
- `agentskill generate <repo>` is the fresh-draft path; `agentskill update <repo>` is the in-place merge/preservation path.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [airscripts/agentskill](https://github.com/airscripts/agentskill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
