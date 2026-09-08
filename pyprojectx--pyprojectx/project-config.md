---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to coding agents when working with code in this repository.

## What this is

Pyprojectx executes commands and aliases defined in the `[tool.pyprojectx]` section of a
`pyproject.toml`, installing the required tools on-the-fly into isolated, per-project virtual
environments. Think "Gradle wrapper for Python": a small committed `pw` script bootstraps
everything, so a freshly cloned project can be built with no prior tool installation beyond a
Python 3.9+ interpreter.

**This project dogfoods itself**: development tasks are run through `./pw <alias>` (see aliases
in `pyproject.toml`).

## Common commands

**Everything goes through `./pw`.** Never invoke `uv`, `pytest`, `ruff`, or `ty` from the system
PATH — a system copy is a different version than the one this project pins, and it runs outside the
tool context. Prefix every `uv` call with the wrapper (`./pw uv sync`, `./pw uv pip list`): `uv` is
a requirement of the `main` tool context, so `./pw uv …` runs the project's own uv. `pytest`, `ruff`
and `ty` are reached through their aliases below, or via `./pw uv run <tool>`.

Aliases are defined in `[tool.pyprojectx.aliases]`.

```bash
./pw install          # uv sync — create .venv with dev dependencies
./pw test             # unit + integration tests
./pw unit-test        # uv run pytest tests/unit
./pw integration-test # uv run pytest tests/integration (slow: creates real venvs, downloads tools)
./pw lint             # ruff check
./pw format           # ruff format + import sort
./pw check            # lint + test
./pw build            # install + check + uv build
./pw clean            # remove .venv, dist, caches
```

Run a single test — aliases append extra args, so pass a path or `-k` filter to `unit-test`:

```bash
./pw unit-test tests/unit/test_config.py    # narrow to one file
./pw unit-test -k camel                      # filter by name
./pw uv run pytest tests/unit/test_config.py -k camel   # directly, after ./pw install
```

(The `run` alias is `uv run pyprojectx …` — it invokes pyprojectx itself, not arbitrary commands.)

`./pw --info` lists all available aliases, scripts, and tool contexts. `./pw --info <name>`
shows what a specific command resolves to.

## Two entry points, one source

- `src/pyprojectx/wrapper/pw.py` is the **wrapper** (the bootstrap). It is dependency-free
  (stdlib only), creates the `.pyprojectx/pyprojectx-<version>` venv, installs the `pyprojectx`
  package into it, then delegates to it. `VERSION`/`UV_VERSION` are `__version__`/`__uv_version__`
  placeholders substituted at release time.
- `src/pyprojectx/cli.py` (`main`) is the **installed package** entry point that the wrapper
  invokes. All real logic lives here and in the sibling modules.
- The root `./pw` (and `pw.bat`, `pw.ps1`) are the committed copies of the wrapper with versions
  filled in, used to develop this repo itself. `tests/conftest.py` copies the *source* wrapper
  into temp projects and sets `PYPROJECTX_PACKAGE` to point back at this checkout (editable
  install) for integration tests.

**Never edit the root `pw`, `pw.bat` or `pw.ps1`.** They are generated copies: wrapper changes go
into `src/pyprojectx/wrapper/pw.py` only, and the root copies are refreshed after a new version is
released (by `bin/prep-release.py`, or with `./pw --upgrade` once the release is out).

When editing the wrapper, keep it stdlib-only — it must run before anything is installed.

## Architecture

Flow of a command `./pw <cmd> <args>`:

1. **`wrapper/pw.py`** parses global options, ensures the pyprojectx venv exists, re-invokes the
   installed `pyprojectx` console script with the same args plus resolved `--toml`/`--install-dir`.
2. **`cli.py::_run`** dispatches on options (`--add`, `--install-context`, `--lock`, `--info`,
   `--clean`) or resolves `<cmd>` in priority order:
   - **alias** or **script** match (`config.find_aliases_or_scripts`) → `_run_alias_cmds`
   - otherwise run in a **tool context** (`config.get_ctx_or_main`) → `_run_cmd_in_ctx`, falling
     back to the `main` context.
3. **`config.py::Config`** parses `[tool.pyprojectx]`. It separates **tool contexts** (named sets
   of requirements) from **aliases** and settings (`env`, `cwd`, `shell`, `scripts_ctx`,
   `prerelease`, `lock-python-version`). `[tool.pyprojectx.os.<platform>]` blocks are merged in
   per-platform via `_merge_os_config`.
4. **`env.py::IsolatedVirtualEnv`** creates and runs commands inside a venv, using **uv** (`uv venv`,
   `uv pip install`). The venv path is derived from an md5 **hash of the requirements**
   (`hash.py`), so different requirement sets get different venvs and a change triggers reinstall.
   Installed tool scripts are symlinked/copied into `.pyprojectx/<ctx>/`.
5. **`lock.py`** freezes requirements to `pw.lock` via `uv pip compile --universal`. On each run,
   `get_or_update_locked_requirements` compares the requirements hash against `pw.lock` and uses
   locked versions when they match. Editable installs and contexts with a custom `dir` can't be
   locked (`can_lock`).
6. **`requirements.py`** implements `--add` (append a package to a context in `pyproject.toml`,
   after verifying it installs).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pyprojectx/pyprojectx](https://github.com/pyprojectx/pyprojectx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
