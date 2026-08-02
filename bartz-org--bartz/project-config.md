---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Project

bartz (BART vectoriZed) — a fast implementation of Bayesian Additive Regression Trees (BART) in JAX. Trees are stored as heap arrays for efficient vectorized operations via `jit`/`vmap`/`lax.scan`.

## Commands

All our development commands are make targets. All make targets use `uv run` under the hood.

## Directory layout

We often use a worktree-first layout where the directory bartz/ is not a worktree, while each subdirectory bartz/foo, bartz/bar, etc. is a worktree. When started from a subdirectory, stay there, and don't try to read/access stuff outside of the worktree.

## Workflow

To check the code you write:
- `make lint`
    - cheap to run, unleashes all linters on everything
    - don't show your work without this first!
- when changing/writing documentation for public stuff:
    - run `make docs`
    - check the html documentation is fine
        - the stuff that breaks most often is type hints, return ones in particular
- `make setup` if needed
    - this sets R up and checks python and jax work fine (incl. if jax picks up any gpu)
    - cheap to run (cached), does not clean, idempotent
    - use liberally if it looks like R is not working
- run the unit tests relevant to your code changes with `uv run pytest ...`
    - not all tests right away because the full test suite takes a long time to run
- at the end of debugging, run the full test suite to check everything works
    - use `uv run pytest`; we have a `make tests`, but its config is pretty heavy on a laptop and blocks other agents/people working in parallel
    - skip this if you think the focused tests were sufficient for a surgical change
- do not update changelog, we write it before release

## Architecture

**Source layout:** `src/bartz/`

| Module | Role |
|---|---|
| `_interface.py` | `Bart` class — high-level public API |
| `BART/` | R BART3-compatible wrappers (`mc_gbart`, `gbart`). The purpose of `mc_gbart` is to maintain a stable interface matching the R BART3 package; when modifying the library internals, adapt `mc_gbart`'s implementation to fit while preserving its external interface. |
| `stochtree/` | stochtree-compatible wrapper `BARTModel`, stable compatibility interface like BART3 |
| `mcmcstep/` | MCMC state (`State`, `Forest`, `StepConfig`), `init`, `step` |
| `mcmcloop/` | MCMC loop orchestration (`run_mcmc`, `evaluate_trace`) |
| `grove/` | Decision tree operations on heap arrays (leaves, splits, traversal) |
| `prepcovars/` | Covariate preprocessing (binning, standardization, R format parsing) |
| `_jaxext/` | JAX utility extensions (vmap, dtypes, device helpers, `scipy/` subpackage) |
| `debug/` | Trace validation, prior sampling, R↔bartz tree conversion |
| `testing/` | `DGP` and `gen_data` for synthetic datasets |

**Data flow:** covariates → `prepcovars` → `mcmcstep.init` → `mcmcloop.run_mcmc` (calls `mcmcstep.step` per iteration) → `RunMCMCResult` with posterior tree samples

State objects are immutable `equinox.Module` dataclasses. Multi-device parallelism via `jax.sharding`.

Interface hierarchy:
- compatibility wrappers `mc_gbart`, `gbart`, `BARTModel`
    - main user interface `Bart`
        - MCMC setup `init()`, MCMC runner `run_mcmc()`
            - MCMC step `step()`

## VCS style

- **Commit messages**
    - <= 50 characters wide description
    - <= 72 characters wide body
    - conventional commits style
- **PR description**
    - study the diff, don't just trust the commit messages (since stuff may have been reverted or altered over the course of development)
    - reason about what is the main purpose of the PR and open with that
        - understanding a description is easier if it opens with the point
        - don't just list all changes mechanically
    - keep it short, short, short
    - no hard wrap, each paragraph on one physical line (github UI preserves source newlines, let it soft wrap instead)
    - don't include a "test plan". you are not going to actually follow it anyway
    - remember to include your "Generated with Claude..." footer

## Code style

- **Formatter/linter:** ruff with single quotes
- **Imports:** generally use `from foo import bar` (relative import) instead of `import foo; foo.bar`
    - but for some heavily used big (sub)modules, e.g., `from jax import random; random.foo` is preferred to `from jax.random import foo, foo1, foo2, ..., foo999999`.
- **Headers** All source files carry an MIT copyright header
    - when creating a new file, use only the current year for the copyright notice
- **docstrings:**
    - numpy convention
    - class attributes documented individually with string just below (not in class docstring)
        - but not global variables
    - keep docstrings short, don't fill them with implementation details
        - related: no redundant comments, if the code is readable, it's self-documenting
        - docstrings and comments shall be _timeless_, not a narration of the development work
        - again, BE BRIEF, humans read slower than you!
    - keep private/internal docstrings short or absent if they are so already

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bartz-org/bartz](https://github.com/bartz-org/bartz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
