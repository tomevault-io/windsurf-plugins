---
trigger: always_on
description: Agent workflow for working in this repo. For codebase design and conventions, read [ARCHITECTURE.md](ARCHITECTURE.md) before core code changes. Do **not** load [CONTRIBUTING.md](CONTRIBUTING.md) by default — it is a long human-contributor guide (setup, permissions, PR etiquette). Consult it only when the task is explicitly about contributor workflow or onboarding humans.
---

# AGENTS

Agent workflow for working in this repo. For codebase design and conventions, read [ARCHITECTURE.md](ARCHITECTURE.md) before core code changes. Do **not** load [CONTRIBUTING.md](CONTRIBUTING.md) by default — it is a long human-contributor guide (setup, permissions, PR etiquette). Consult it only when the task is explicitly about contributor workflow or onboarding humans.

## Environment

Use `mamba`, `micromamba`, or `conda` (in that preference order) to manage the `CausalPy` environment. Reuse an existing `CausalPy` env whenever possible; do not create or update an env unless the task needs the project environment and the existing env is missing, stale, or broken. Use `$CONDA_EXE run -n CausalPy <command>` for commands that import project code, run tests, build docs, or use repo tooling; never use `$CONDA_EXE activate`. For simple text/JSON inspection helpers that do not import project code, any Python on `PATH` is fine.

See the [python-environment skill](.agents/skills/python-environment/SKILL.md) for full setup instructions: tool detection, environment creation, editable install, and troubleshooting.

- If `$CONDA_EXE run -n CausalPy ...` fails because the named env cannot be resolved, inspect `$CONDA_EXE env list` and retry with `$CONDA_EXE run -p <full-prefix> <command>`.
- In git worktrees, prefer reusing an existing env. Because the repo uses editable installs, rerun `make setup` in the current worktree only when that checkout has not been installed into the env yet or when dependencies changed.

- Dependencies live in `pyproject.toml`; `environment.yml` is generated from it by a prek hook (do not edit by hand). Optional: `pymc-marketing` is in the `docs` extra only.
- **Development**: The supported setup is the conda env (`environment.yml`). `pip install -e .[dev]` works but does not include conda-only tooling (e.g. `make`, `pymc-bart`, `marimo`); do not suggest pip-only dev as equivalent.

## Testing

- Write all Python tests as `pytest` style functions, not unittest classes
- Use descriptive function names starting with `test_`
- Prefer fixtures over setup/teardown methods
- Use assert statements directly, not self.assertEqual
- Never create throwaway test scripts or ad hoc verification files
- If you need to test functionality, write a proper test in the test suite
- All tests go in the `causalpy/tests/` directory following the project structure
- Tests should be runnable with the rest of the suite (`python -m pytest`)
- Even for quick verification, write it as a real test that provides ongoing value
- Preference should be given to integration tests, but unit tests are acceptable for core functionality to maintain high code coverage.
- Tests should remain quick to run. Tests involving MCMC sampling with PyMC should use custom `sample_kwargs` to minimize the computational load.

## Sandbox and permissions

- **PyMC/PyTensor require filesystem access** outside the workspace (e.g. `~/.pytensor/compiledir_*` for C compilation cache, `~/.matplotlib` for font cache). The default Cursor sandbox blocks writes to these paths, causing misleading `ValueError` or `PermissionError` failures that look like real test errors but are not.
- **Always use `required_permissions: ["all"]`** when running `pytest`, `make doctest`, or any command that imports PyMC, PyTensor, or matplotlib. This avoids false negatives from sandbox restrictions.
- **If a test run shows `compiledir ... you don't have read, write or listing permissions`**, that is a sandbox problem, not a code problem. Re-run with `["all"]` permissions before investigating further.

## Documentation

- **Structure**: Notebooks (how-to examples) go in `docs/source/notebooks/`, knowledgebase (educational content) goes in `docs/source/knowledgebase/`
- **Notebook naming**: Use lowercase hyphen-separated words with the full method name spelled out, then the dataset/variant token (if any), then the backend (`pymc` or `sklearn`). Pattern: `{method}[-{variant}]-{backend}.ipynb` (e.g., `difference-in-differences-pymc.ipynb`, `regression-discontinuity-drinking-sklearn.ipynb`, `synthetic-control-brexit-pymc.ipynb`). Prefer descriptive words over acronyms for SEO: `interrupted-time-series` over `its`, `regression-discontinuity` over `rd`, `synthetic-control` over `sc`, `instrumental-variables` over `iv`. When renaming an existing notebook, update `docs/source/notebooks/gallery.yaml`, run `make gallery`, add an entry to `rediraffe_redirects` in `docs/source/conf.py`, and never remove older redirect keys.
- **MyST directives**: Use `:::{note}` and other MyST features for callouts and formatting
- **Glossary linking**: Link to glossary terms (defined in `glossary.rst`) on first mention in a file:
  - In Markdown files (`.md`, `.ipynb`): Use MyST syntax `{term}glossary term``
  - In RST files (`.rst`): Use Sphinx syntax `:term:`glossary term``

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pymc-labs/CausalPy](https://github.com/pymc-labs/CausalPy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
