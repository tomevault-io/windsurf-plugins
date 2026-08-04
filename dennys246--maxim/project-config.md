---
trigger: always_on
description: Maxim is a bio-inspired cognitive architecture for AI agents. It combines a 5-agent pipeline (Perception, Memory, Exec, Goal, Statistician) with biological memory systems (Hippocampus, ATL, Angular Gyrus, SCN, NAc) and a reactive Default Network. Works headless, in simulation, or connected to a robot.
---

# CLAUDE.md

## Project Overview

Maxim is a bio-inspired cognitive architecture for AI agents. It combines a 5-agent pipeline (Perception, Memory, Exec, Goal, Statistician) with biological memory systems (Hippocampus, ATL, Angular Gyrus, SCN, NAc) and a reactive Default Network. Works headless, in simulation, or connected to a robot.

## When making changes — required checks

Run these before considering any non-trivial task done:

```bash
# Lint + format
ruff check src/ tests/
ruff format src/ tests/

# Tests (fast suite)
python -m pytest tests/ -x -q -m "not slow" --ignore=tests/integration/test_memory_hub.py

# If touching memory/, decisions/, integration/memory_hub.py:
python -m pytest tests/integration/test_memory_hub.py -q
```

Additional guardrails:
- **Test interactive changes with logging.** When touching interactive mode (display, prompts, stdin reader, orchestrator sim loop), capture a session with `MAXIM_LOG_FILE=/tmp/maxim.jsonl maxim --sim "test basic recall" --interactive --sim-max-turns 3` and read the JSONL to verify percepts, tool calls, and followups flow correctly. Check for `ACTION_FOLLOWUP` entries to confirm user responses reach the LLM. Use `MAXIM_BACKEND_TRACE=1` for per-call token/latency data.
- **No band-aid fixes.** If you spot a bug while working on a task, determine whether the fix addresses the root cause or merely hides the symptom. If it's the latter — a special case, a swallowed exception, a flag that toggles around broken behavior, a fix that would need to be repeated elsewhere — stop, describe the root cause and the scope of the proper fix, and ask the user how to proceed. Never silently choose the smaller fix because it's easier.
- Prefer editing existing modules over creating new ones — this codebase favors many small files already
- Don't rename bio-system classes (Hippocampus, ATL, NAc, SCN, EC, AngularGyrus) — names are load-bearing for the mental model
- If you touch provenance, run a sim with `MAXIM_PROVENANCE_VERBOSITY=2` and eyeball the trace
- **Run `mypy` on public API files** after changing api.py, session.py, create.py, load.py, or __init__.py: `mypy src/maxim/__init__.py src/maxim/api.py src/maxim/session.py src/maxim/create.py src/maxim/load.py --ignore-missing-imports --follow-imports=silent` (same invocation CI runs — `--follow-imports=silent` scopes errors to the five files; the rest of the codebase is not mypy-clean yet)
- **Run `ruff format`** after any changes: `ruff format src/ tests/`
- **Parallel sessions use worktrees.** When ≥2 Claude sessions run concurrently on independent work, each uses its own git worktree (`git worktree add ../Maxim-wt-<branch> -b <full-branch>`) and works entirely in absolute paths within that worktree. Single-session work stays in the main checkout. Tests in worktrees need `PYTHONPATH=src`. Note `~/.maxim/` is shared across worktrees — don't run sims from concurrent doc/code sessions or they'll collide on persisted state.

## Lessons learned (bugs that bit us)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dennys246/Maxim](https://github.com/dennys246/Maxim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
