---
trigger: always_on
description: Rules and conventions for AI agents working in this repo. Claude Code,
---

# AGENTS.md

Rules and conventions for AI agents working in this repo. Claude Code,
Codex, Cursor, and similar tools should treat these as overriding
defaults. Layered atop these are per-subdirectory `AGENTS.md` files
(`experiments/AGENTS.md`, `models/AGENTS.md`, …) which add subsystem-
specific rules.

## Project shape

MarinFold trains protein-structure language models on Marin
infrastructure. Concerns at the repo root:

- `experiments/` — one dir per GitHub issue tagged `experiment`.
  All new work starts here as `exp<N>_<kind>_<name>/`. Holds prose
  READMEs, launchable `.py` files, and small artifacts (CSVs feeding
  plots, plots themselves).
- `scripts/` — repo-management scripts (`scaffold.py`, `itemize.py`,
  `history.py`). Run with plain `python scripts/<name>.py`.
- `marinfold/` — the top-level Python package. Backends, model
  registry, document-structure shared toolkit, the
  document-structure impls (as subpackages of
  `marinfold.document_structures.<name>`), and the user-facing
  `marinfold infer` / `marinfold evaluate` CLI.
- `models/` — library for model-training experiments
  (`marinfold_models.defaults`, `marinfold_models.simple_train_config`,
  …). Kept separate from `marinfold/` because the training dep
  stack (marin/levanter/jax) is heavy and platform-coupled and
  has no business in the inference path.

Kind libraries for `evals` and `data` (and any other future kind)
are created on demand — when a second experiment in that kind
needs the same helper. Don't pre-scaffold an empty library.

Each kind dir / top-level Python package is a self-contained
directory with its own `pyproject.toml` and its own `.venv`.

Experiments may import from any kind library via path deps in their
own `pyproject.toml`. Libraries DO NOT import from experiments — that
direction is forbidden. If two experiments need the same helper,
promote it to the kind library once a second use case actually exists
(not before).

Experiment dirs are never copied into a kind dir — there is no
graduation step. Reusable code lands in the kind library from the
start and the experiment imports it; the experiment dir keeps only
its own driver code and results.

See `experiments/README.md` for the workflow,
and `marinfold/README.md` for the inference/CLI surface. For any
data-generation pipeline that runs on the marin Iris cluster via
Zephyr, read the [`zephyr-pipeline-performance`](.agents/skills/zephyr-pipeline-performance/SKILL.md)
skill before drafting `cli.py`. It captures the handful of decisions
that separate a fits-in-budget run from an overnight one.

## Shared coding practices

Mirrored from `marin-community/marin-experiments/AGENTS.md` — keep them
consistent unless we deliberately diverge.

### Tooling

- Assume Python >= 3.11.
- Always use `uv run` for Python entry points. If that fails, try
  `.venv/bin/python` directly.
- Use type hints.
- Prefer `pyrefly` for type-checking.

### Communication & commits

- NEVER SAY "You're absolutely right!"
- Never credit yourself in commits. NEVER EVER EVER credit yourself in
  commit messages.

### Code style

- Put all imports at the top of the file. Avoid local imports unless
  technically necessary (e.g. to break circular dependencies or guard
  optional dependencies).
- Prefer top-level functions when code does not mutate shared state;
  use classes to encapsulate data when that improves clarity.
- Prefer top-level Python tests and fixtures.
- Disprefer internal mutation of function arguments, especially config
  dataclasses. Prefer returning a modified copy
  (`dataclasses.replace(...)`) so call sites stay predictable.
- Use early returns (`if not x: return None`) when they reduce nesting.
- Do not introduce ad-hoc compatibility hacks like
  `hasattr(m, "old_attr")`; update the code consistently instead.
- Do not use `from __future__ import ...` statements.
- Document public APIs with concise Google-style docstrings.

### Error handling

- Let exceptions propagate by default.
- Only catch exceptions when you can add meaningful context and re-
  raise, or when you are intentionally altering control flow.
- NEVER EVER SWALLOW EXCEPTIONS unless specifically requested.

### Deprecation

**No backward compatibility**: do not add deprecation warnings,
fallback paths, or compatibility shims. Update all call sites instead.
Only add backward compatibility if the user explicitly requests it.

### Comments

Write detailed comments when describing behavior as a whole, e.g. at
module or class level, or when describing some subtle behavior.
Do not generate comments that merely restate the code.

### Testing

- Always fix tests if you broke them.
- Do not fix tests by relaxing tolerances or hacking around them.
- Avoid "tautological" tests that merely restate implementation logic.
- Run the appropriate tests for your changes.

## Hard rules

### Branch + PR for substantive work; don't push directly to main

Substantive changes — new code, multi-file edits, design decisions —
go on a feature branch and land via a GitHub PR, even when the
intent is to merge straight into `main`. The branch doesn't need to
live long: open the PR, run review (e.g. `/ultrareview` against
`origin/main`), merge, delete the branch.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Open-Athena/MarinFold](https://github.com/Open-Athena/MarinFold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
