---
trigger: always_on
description: Poe the Poet is a task runner for Python projects that integrates with poetry and uv.
---

# Agent Guide

Poe the Poet is a task runner for Python projects that integrates with poetry and uv.

## Use poe tasks

This project uses poethepoet to manage development tasks. Prefer to use poe tasks when available to reduce friction.

Run `poe` to see available tasks and their descriptions.


```bash
poe check             # run all quality checks (style, types, lint, tests) - this takes a while.
poe test              # run full test suite
poe test-quick        # skip slow/flaky tests
poe format            # auto-format code (ruff + black)
poe lint              # ruff linting only
poe types             # mypy type checking
```

**Tip:** cmd tasks (like test or test-quick) pass extra args to the underlying command, so you can run specific tests:
```bash
poe test tests/test_script_tasks.py -k "test_running"
poe test-quick -x  # stop on first failure
```

### Testing

Run tests with:

```sh
poe test [extra pytest arguments]
```

Reference tests/README.md for instructions on how to write, run, and debug tests in this project.

## Key Patterns

**Task types** use a metaclass registry (`task/base.py:MetaPoeTask`). Each task type:
- Inherits from `PoeTask`
- Sets `__key__` and `__content_type__` class attributes
- Implements `_handle_run()` for execution logic

**Executors** follow the same pattern (`executor/base.py:MetaPoeExecutor`).

**Config loading** is async-first. See `config/config.py:PoeConfig`.

**Type hints** are used throughout. Circular imports are avoided with `TYPE_CHECKING` guards.

**Lazy imports** are used strategically with the goal of reducing latency, since this is a CLI app. Note the distinction: imports only needed for type annotations belong in a `TYPE_CHECKING` block (with `from __future__ import annotations` at the top of the module), not as lazy runtime imports inside functions.

## Where to Look

| To do this...                     | Look here                          |
|-----------------------------------|------------------------------------|
| Add a new task type               | `task/base.py`, then `task/*.py`   |
| Add a new executor                | `executor/base.py`                 |
| Modify CLI behavior               | `ui.py`, `app.py`                  |
| Change config parsing             | `config/config.py`                 |
| Add environment variable handling | `env/manager.py`                   |
| Fix shell completion              | `completion/`                      |
| Edit the bundled Agent Skill      | `poethepoet/skills/poethepoet/`    |

## Bundled Agent Skill

This project ships an Agent Skill at `poethepoet/skills/poethepoet/` that teaches AI coding assistants how to use poe. End-users install it via `poe _install_skill`, which copies the tree into a detected `.claude/skills/`, `.codex/skills/`, etc. (see `poethepoet/skills/install.py`).

- **Skill content**: `SKILL.md` + `references/*.md` (task-types, task-options, args-reference, creating-tasks, task-packages).
- **Pinned version**: `poethepoet/skills/poethepoet/version.txt` — kept in step with poe's user-facing surface. Bump it when skill content changes meaningfully.
- **Evals**: `tests/skills/evals.json` (corpus) and `tests/skills/run_evals.py` (runner). The runner copies the skill into a fixture project's `.claude/skills/` and shells out to `claude -p`, so it exercises the real discovery path. Run via `poe eval-skill`.

When you change poe's behaviour, syntax, or user-facing semantics, check whether the skill's reference docs still describe it correctly — drift here causes downstream agents to hedge or generate broken configs. If you spot a gap while working on the skill, the canonical answer is in the installed source (`poethepoet/task/*.py`) and the published docs (https://poethepoet.natn.io); verify before paraphrasing.

## Test Fixtures

Tests use fixture projects in `tests/fixtures/*_project/`. The `run_poe` fixture is the main way to invoke poe in tests, or `run_poe_subprocess` if necessary - see `tests/conftest.py`.

## Branching

- Branch from `development` for features/bugfixes
- Branch from `main` only for docs or hotfixes

## Development process

- The user docs are in the repo and good source of context on how the product is meant to work.
- When writing tests, observe and replicate local testing style.
- When introducing new patterns or making any kind of contribution to architecture present the plan for operator review before making change to code.

### Code style

- Don't use single character variable names
- Do use the walrus operator whenever applicable
- docstrings must always use three lines minimum: opening `"""` alone on its own line, content, closing `"""` alone on its own line — never `"""text"""` on one line
- Use **relative imports** for everything inside the `poethepoet` package — `from ..task.base import PoeTask`, not `from poethepoet.task.base import PoeTask`. Applies to top-level, `TYPE_CHECKING`, and lazy-inside-function imports alike. Sole exception: `poethepoet/__main__.py` keeps `from poethepoet import main` so it works whether invoked via `python -m poethepoet` or as a direct script.
- This is a CLI, so be mindful of performance concerns
- `poe lint` and `poe types` must pass
- `poe format` should be run to ensure correct formatting

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nat-n/poethepoet](https://github.com/nat-n/poethepoet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
