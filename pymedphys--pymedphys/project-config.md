---
trigger: always_on
description: Guidance for every coding agent working in this repository. Follow it, and
---

# AGENTS.md

Guidance for every coding agent working in this repository. Follow it, and
the [contributor language policy](CONTRIBUTING.md#language), for every change.

## Development Commands

### Setup and Installation

```bash
# Install with uv (required for development)
uv sync --python 3.14 --locked

# Install pre-commit hooks
uv run -- pre-commit install

# Install for user use only
python -m pip install "pymedphys[user]"
```

### Testing

```bash
# Run default tests (slow/database selection has separate options)
uv run -- pymedphys dev tests

# Run specific test file or directory (relative to lib/pymedphys or the cwd)
uv run -- pymedphys dev tests tests/path/to/test.py

# Run with specific pytest options
uv run -- pymedphys dev tests -v -s -k "test_name"

# Add the slow tests to the default selection, or run only the slow tests
uv run -- pymedphys dev tests --include-slow
uv run -- pymedphys dev tests --slow

# Run tests in parallel, one worker per CPU (pytest-xdist)
uv run -- pymedphys dev tests --slow -n auto

# Run doctests
uv run -- pymedphys dev doctests
```

### Code Quality

```bash
# Run linting with ruff (automatically fixes issues)
uv run -- ruff check --fix .
uv run -- ruff format .

# Run type checking with pyright
uv run -- pyright

# Run pre-commit on all files
uv run -- pre-commit run --all-files

# Check imports are clean
uv run -- pymedphys dev imports
```

### Documentation

```bash
# Build documentation
uv run -- pymedphys dev docs

# The docs use Jupyter Book and are located in lib/pymedphys/docs/
```

Documentation notebooks must use declared, locked dependencies rather than
installing packages while running. Add documentation dependencies to the
`docs` dependency group and regenerate `uv.lock`; CI and ReadTheDocs both
install the documentation environment from it.
Unexpected notebook errors and documentation build warnings fail the build.
An install cell kept for readers running a notebook elsewhere (for example on
Colab) must carry the `skip-execution` cell tag. Sphinx configuration is
generated into `lib/pymedphys/docs/conf.py` from `_config.yml`; the generated
file is gitignored, so edit `_config.yml`.

Write procedures as instructions with their success criteria. State each
fact once and link to it, prefer fixing a defect over documenting a workaround
for it, and keep incident history and evidence caveats on the pull request
rather than in the guide. Open a long procedure with a short checklist for
readers who already know it, and move one-time setup into an appendix.

Documents, decision logs, changelog entries, and pull request descriptions
describe the state being merged. Fold revisions made while a pull request is
open into the text; do not record them as superseded decisions, review logs,
or construction history.

Write changelog entries as informative release notes: state what changed, its effect, and what users should do, in declarative sentences. Do not phrase entries as answers to review questions or pre-empt objections ("is not limited to ...", "does not use ...", "has not been measured"); state the scope directly instead. When a fix changes results that earlier versions returned without an error, open the release section with a warning that says which inputs and functions were affected, by how much, and what to re-check, and also list the changed results and any changed return shapes or array order under (Potentially) breaking changes.

Use ordinary Markdown links in Markdown pages and notebook Markdown cells.
Follow the relative source-path and published-URL guidance in
[Writing portable links](lib/pymedphys/docs/contrib/info/docs-guide.rst#writing-portable-links).

Historical, site-specific deployment pages (for example the iCom listener,
tunnelling, and rsync how-tos) record what was done at the time. Do not
modernise, correct, or test their commands, versions, or links. Keep their text
as originally written, or at the latest as it stood before the September 2025
switch from Poetry to uv, and confine changes to a note marking the page as a
historical record.

## Architecture Overview

### Project Structure

PyMedPhys is a medical physics library organised as follows:

- **lib/pymedphys/**: Main library code
  - `_<module>/`: Private implementation modules (e.g., `_dicom/`, `_gamma/`, `_mosaiq/`)
  - `<module>.py`: Public API modules that expose the private implementations
  - `cli/`: Command-line interface implementations
  - `tests/`: Test suite organised by module
  - `_experimental/`: Experimental features not yet stable
  - `_streamlit/`: Streamlit web app components

### Project Maintainers

PyMedPhys is maintained by:

- SimonBiggs
- sjswerdloff
- Matthew-Jennings
- pchlap

Use this list wherever metadata needs the maintainers.

### Key Architectural Patterns

1. **Private/Public Module Pattern**: Implementation details are in `_module/` directories, with public APIs exposed through `module.py` files at the package root.

2. **Delivery Abstraction**: The `Delivery` class (`_base/delivery.py`) provides a unified interface for treatment delivery data from various sources (DICOM, TRF files, Mosaiq, Monaco).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pymedphys/pymedphys](https://github.com/pymedphys/pymedphys) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
