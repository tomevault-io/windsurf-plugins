---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`groundhog-researcher` is an LLM-powered iterative function-optimization framework. A user defines a `Task` (data + context + evaluator), an optimizer iterates with one or more `Strategy` instances, and every `Attempt` is persisted in an immutable tree under `attempts/`.

The package is published to PyPI as `groundhog-researcher` and exposes the `groundhog` / `ghg` CLI for scaffolding tasks.

## Common commands

```bash
# Test suite (~336 tests, fast — runs in under a minute)
pytest

# Iterating on a single area (preferred during work)
pytest tests/test_concepts.py -x          # core primitives — fastest, broadest coverage
pytest tests/test_agent_concepts.py        # agent backends, AgentStrategy, sandbox behavior

# A single test
pytest tests/test_concepts.py -k <name>

# CLI (after installing the package or running from source)
groundhog backends                        # show discovered backends and tier assignments
groundhog init my_task                    # scaffold a task (installs the session skills too)
groundhog prefer <backend>                # preference written to ~/.groundhog/config.json
groundhog attempt <sub>                   # manual attempt lifecycle -- new (--fresh), list,
                                          #   show, commit (gated, --strategy), abort, ...
groundhog eval <path-or-id>               # score a solution dir, .py file, or attempt
groundhog tool list|run                   # run toolkit tools (check-gates is a default)
groundhog skills install [dir]            # copy the packaged session skills into a run dir

# Sandbox probe — runs each agent backend through a 9-op gauntlet, reports filesystem ground truth.
# Use before changes that touch agent sandboxing. Costs API/CLI calls.
uv run tools/probe_agents.py              # all available backends
uv run tools/probe_agents.py codex_cli    # one backend

# Live MNIST smoke (NOT a pytest test — costs money + time, run by user, not automatically)
# Use when changes touch strategies/, optimizers/, agents/, or tools/attempt_log*.py.
cd tests/e2e_mnist_agent && uv run task.py claude 1   # <backend> <iterations>; backends: claude, copilot, codex
cd tests/e2e_mnist_agent && uv run task.py llm 4      # LLM-strategy rotation (Improve/Cross/Fresh) on the cheapest available auth
```

## Design principles (check new code against these)

- **base/ is interfaces only.** No implementation in base/; base classes are the essence of the concept, nothing more.
- **Strategies own the full loop.** Select prior → workspace → generate → evaluate → record. Strategy-specific logic never leaks into the optimizer.
- **Composed method pattern.** Strategy `__call__` reads like a story of named step methods; details live in the steps.
- **Raw results, never scores.** Attempts store metrics dicts; scoring is read-side via per-stage scorers. Never persist a score in the attempt record — the one sanctioned carve-out is the mutable score NOTE (notes.json / git note), a display-only cache that is never read for decisions.
- **Toolkit is capabilities, not tools.** Strategies `hasattr`-check and fall back gracefully; never assume a capability exists.
- **Config is self-documenting.** Every knob is `param(default, "description")`, introspectable via `Config.describe()`.
- **Nothing is discarded.** Every attempt — success or failure — is recorded; failures inform future strategies.
- **Trunks are derived, not stored.** Apply a scorer to the tree to get trunks; never persist trunk membership.

## Releasing

The version lives in ONE place: `src/groundhog/__init__.py` `__version__`
(`pyproject.toml` is hatch-dynamic and reads it). The publish workflow's
version-check job rejects a tag that doesn't equal `__version__`.

Push a tag `v<version>` to trigger the GitHub Actions trusted-publishing workflow that releases to PyPI (it reruns the full test matrix first). Every release is a discrete decision — confirm with the user before commit/push/tag.

Full pre-release checklist: `checklist.md` (repo root) — the consistency manifest covering everything tests can't catch (templates, README, vault alignment, model IDs). Run it before any release commit. CI runs the whole suite via `pytest tests/` — new test files are auto-discovered, no workflow edits needed.

## Architecture

Four layers, each implemented in its own subpackage. `base/` defines interfaces only; everything else is implementations a user can subclass or swap.

```
USER CODE  →  OPTIMIZER  →  STRATEGY  →  AGENT BACKEND  →  AGENT CLI
                                       (or LLM BACKEND for non-agent strategies)
```

### `base/` — interfaces

- **`Task`** = `Data + Context + Evaluator`. The problem definition.
- **`Evaluator.get_stages()`** returns ordered `EvalStage`s (cheap → expensive). Each stage has a `scorer` callable that maps `StageResult.metrics` to a float — **scores are computed, not persisted**, so changing the scorer reinterprets history without re-running anything.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [frizzerdk/groundhog-researcher](https://github.com/frizzerdk/groundhog-researcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
