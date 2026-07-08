---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> `AGENTS.md` symlinks to `CLAUDE.md`, and `.agents/skills/` symlinks to
> `.claude/skills/`. The `treebox` skill's canonical copy lives in `skills/`
> (kept there for users to copy-paste); `.claude/skills/treebox/` symlinks to it.
> Edit the sources — `CLAUDE.md` and `skills/` — and the symlinks reflect the change.

## What this is

`treebox` is a Python CLI that hands AI coding agents isolated, ready-to-run git
worktrees — one directory per worktree *name* (user slug or generated petname),
run either host-native or inside a docker sandbox. It is orchestration glue:
it shells out to `git` and `docker`. Requires **Python 3.11+** (the sandbox
template it provisions is separately pinned to CPython 3.14.6).

## Commands

```bash
uv run treebox ...                      # run the CLI from the working tree
uv run --extra dev python -m pytest     # full unit + integration suite
uv run --extra dev python -m pytest tests/test_units.py::test_name   # single test (or -k <pattern>)
uv run --extra dev ruff check src tests # lint
uv run --extra dev ruff format --check src tests   # format check (drop --check to apply)
uv run --extra dev mypy                 # strict type check (config in pyproject.toml)
uv run --extra dev pre-commit install   # install lint/format/strict type-check hooks (see CONTRIBUTING.md)
scripts/golden-diff.sh                  # diff CLI output against tests/golden snapshots
./scripts/validate.sh                   # full gate: lint + typecheck + shell assets + tests + snapshots + smoke
uv pip install -e ".[dev]"              # editable dev environment
uv run --extra docs mkdocs serve        # docs site (docs/ + mkdocs.yml), live-reloading
uv run --extra docs mkdocs build --strict   # build docs to site/ (gitignored)
```

Run `validate.sh` before changes that affect provisioning, runners, git
handling, shell assets, or CLI output; it includes the golden CLI-output
snapshots and shell asset checks. Use `scripts/golden-diff.sh --update` only
after an intentional output change has been agreed. `pytest` is enough for
smaller changes.

## Architecture

The whole tool is organized around one seam: **provision (always host-side) vs.
run (pluggable)** — with three registries that define everything swappable.

**The three-seam glossary** (each term is one axis, one module, one registry):

- **Harness** — *which agent CLI launches* (`claude`, `codex`).
  `harnesses.py`: the `Harness` dataclass + `HARNESSES` registry.
- **Runner / isolation** — *where it executes* (`host`, `docker`).
  `runners/`: the `Runner` protocol + `RUNNERS` registry.
- **Ecosystem** — *what setup runs* (uv, npm, pnpm, go, cargo).
  `ecosystems.py`: the `Ecosystem` dataclass + `ECOSYSTEMS` registry.

Module map:

- **`provision.py`** owns the host-side half, identical for every runner:
  `fetch → resolve branch → git worktree add → install pre-push guard →
  copy submodules → copy .env → runner.setup (cache-backed) →
  record lockfile hash → hand to runner`. An explicit `create NAME` uses the
  name as the branch, created fresh from `origin/<base>` (an existing branch is
  a `BRANCH_EXISTS` conflict — `--checkout` is the resume path); a nameless
  `create` makes a `treebox/<petname>` placeholder. Every worktree gets the
  pre-push guard (`extensions.worktreeConfig` + `core.hooksPath` into the
  private git dir), which rejects `treebox/*` refs — so placeholders are
  un-pushable until renamed, real branches unaffected.
- **`harnesses.py`** is the one place agent-CLI wiring lives: each `Harness`
  hides its autonomous launch argv, host login dir, staged login files, and
  login advice behind a small method interface; `VALID_HARNESSES` and the
  doctor login rows derive from `HARNESSES`. Boundary values (CLI/TOML/state)
  stay `str` and are resolved to the object once, in `cli.py`.
- **`runners/base.py`** defines the `Runner` protocol — the *only* thing that
  differs between modes — implemented by **`runners/host.py`** (setup + agent in
  the worktree shell) and **`runners/docker.py`** (plain `docker build/run`,
  setup via a baked-in `post-create.sh`, agent via `docker exec`; the worktree
  and its git common dir are bind-mounted at their host paths so in-container
  git just works). `runners/__init__.py` holds the `RUNNERS` registry;
  `get_runner` and `VALID_ISOLATION` derive from it. Doctor-facing vocabulary
  (preflight detail, whether a login is a hard gate) lives in `RunnerFacts`,
  not in the run methods; teardown options (docker's `remove_volumes`) arrive
  at the runner's constructor, never through the protocol.
- **`cli.py`** (Typer) is the entry point: `create [NAME] / enter <ref> /
  list / teardown <ref>... / template <init|list|path> / doctor / version`
  (`ls`/`rm` are hidden aliases of list/teardown, `template ls` of
  `template list`). The `template` sub-app
  scaffolds and inspects operator-owned sandbox templates via `assets.py`'s
  resolver, so customizing a docker sandbox never needs a `python -c` reach
  into the package. `enter`/`teardown` resolve a ref as
  name → live branch → unique substring (`resolve.py`); ambiguity exits 2.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Seth-Peters/treebox](https://github.com/Seth-Peters/treebox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
