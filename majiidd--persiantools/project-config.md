---
trigger: always_on
description: Development guide for AI coding agents and contributors working on
---

# AGENTS.md

Development guide for AI coding agents and contributors working on
`persiantools`.

This file is for repository work: setup, coding, tests, reviews, releases, and
maintenance. End-user installation and usage examples belong in [README.md](README.md).

When this guide and tool output disagree, the repository configuration is the
source of truth: [pyproject.toml](pyproject.toml), [Makefile](Makefile),
[.pre-commit-config.yaml](.pre-commit-config.yaml), [.flake8](.flake8), and
[.github/workflows/ci.yml](.github/workflows/ci.yml).

## Scope and Precedence

- This root `AGENTS.md` applies to the whole repository.
- Direct maintainer or user instructions in the current task override this file.
- Keep this file agent-focused. Put human-facing project narrative in
  `README.md` and release history in `CHANGELOG.md`.

## Project Snapshot

- `persiantools` is a pure-Python, MIT-licensed package for Jalali (Shamsi)
  dates/datetimes and Persian text utilities.
- Runtime dependencies must stay at zero except for the current Windows-only
  `tzdata` dependency.
- Supported runtimes are CPython 3.9 through 3.14 and PyPy. CI also runs an
  experimental CPython 3.15 job on Linux.
- Code must remain Python 3.9-compatible. Pre-commit enforces this with
  `pyupgrade --py39-plus`.
- The default branch is `master`; releases are published from bare version tags
  such as `6.0.1`.

## Repository Map

| Path | Purpose |
| --- | --- |
| `persiantools/` | Library source code. Public package metadata and `__version__` live in `persiantools/__init__.py`. |
| `persiantools/jdatetime.py` | `JalaliDate` and `JalaliDateTime`, including conversion, parsing, formatting, arithmetic, and timezone behavior. |
| `persiantools/digits.py` | Digit conversion and Persian number-to-word helpers. |
| `persiantools/characters.py` | Arabic/Persian character normalization helpers. |
| `persiantools/utils.py` | Shared validation and conversion helpers. |
| `tests/` | Pytest suite. Test files are named `test_*.py`. |
| `README.md` | End-user documentation and examples. |
| `CHANGELOG.md` | Human-written release notes. |
| `pyproject.toml` | Project metadata, dependency groups, build backend, and tool configuration. |
| `uv.lock` | Locked development dependencies. Regenerate with `make lock`; do not hand-edit. |
| `Makefile` | Canonical development commands. |
| `.pre-commit-config.yaml` | Format, lint, type, security, and metadata validation hooks. |
| `.flake8` | Flake8 configuration kept compatible with Black. |
| `.github/workflows/ci.yml` | CI, build, Test PyPI, and PyPI publishing workflow. |

## Environment

Use `uv` 0.11.0 or newer. For a fresh checkout:

```bash
make install
```

This runs `uv sync --locked` and installs the pre-commit hooks.

Common commands:

| Command | Use |
| --- | --- |
| `make` | Show all documented Make targets. |
| `make sync` | Create or update the local environment from `uv.lock`. |
| `make hooks` | Install pre-commit hooks. |
| `make test` | Run the full test suite. |
| `make cov` | Run tests with terminal coverage details. |
| `make lint` | Run all pre-commit hooks on all files. |
| `make format` | Run isort and Black through pre-commit. |
| `make check` | Run lint and tests, matching the main local CI gate. |
| `make build` | Build the sdist and wheel. |
| `make lock` | Refresh `uv.lock` after dependency metadata changes. |
| `make upgrade` | Upgrade locked dependencies within configured version bounds. |
| `make clean` | Remove local build, cache, and coverage artifacts. |

Useful focused commands:

```bash
uv run --no-sync pytest -ra tests/test_digits.py
uv run --no-sync pytest -ra tests/test_jalalidate.py -k fromisoformat
uv run --no-sync pre-commit run black --files persiantools/jdatetime.py
uv run --no-sync pre-commit run mypy --all-files
```

Prefer `make` targets for final validation because they match the repository's
documented workflow.

## Agent Workflow

1. Start by checking the working tree with `git status --short`.
2. Read the relevant source, tests, and config before editing. Prefer `rg` for
   searches.
3. Preserve unrelated user changes. Do not revert, restage, or clean files you
   did not intentionally modify.
4. Make the smallest coherent change that fixes the issue or implements the
   request.
5. Add or update tests for every behavior change.
6. Update docs only when user-visible behavior, examples, requirements, or
   development workflow changes.
7. Run the narrowest useful test during iteration, then run `make check` before
   finishing when practical.
8. If a required check cannot run locally, report the command and the reason.

## Coding Standards

- Use Black and isort with a 120-character line length.
- Keep public APIs type-annotated and documented with docstrings.
- Match the standard-library `datetime` API where `JalaliDate` or
  `JalaliDateTime` mirrors a `date` or `datetime` behavior.
- Prefer clear, small functions over broad rewrites. Avoid unrelated refactors.
- Keep code compatible with Python 3.9 and PyPy.
- Use only the standard library at runtime unless a maintainer explicitly agrees
  to a dependency change.
- Do not use `eval`.
- Do not add blanket `# noqa` or blanket `# type: ignore` comments.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [majiidd/persiantools](https://github.com/majiidd/persiantools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
