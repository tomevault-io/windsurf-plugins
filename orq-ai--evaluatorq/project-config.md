---
trigger: always_on
description: This file provides guidance to Claude Code when working in `packages/evaluatorq-py`.
---

# CLAUDE.md — evaluatorq-py

This file provides guidance to Claude Code when working in `packages/evaluatorq-py`.

## Parallel sessions

Parallel agent sessions typically run in their own git worktree, so uncommitted
changes you did not make may appear in the working tree from concurrent work.
**Never run `git stash` (any subcommand) or `git reset`.** Not `stash` to clean
the tree, and not `stash pop`/`apply` either: the stash holds other sessions'
autostash entries, and popping one drops a merge into your tree and consumes the
entry. `git checkout <path>` and `git checkout -- .` are equally destructive to
uncommitted work you did not write. When committing, stage only the exact files
your task changed.

The same applies to every subagent you dispatch — say it in the dispatch prompt.
A reviewer that "just needed a clean tree for a moment" has already popped
another session's autostash once.

To read a file as it is on HEAD without touching the tree, use
`git show HEAD:<path>`. To see only your own changes on a shared dirty tree,
diff the paths you touched: `git diff -- <your paths>`.

## Quick Reference

```bash
# Install dependencies (dev group + all optional extras)
uv sync --all-extras --all-groups

# Run unit tests (excludes integration tests)
uv run pytest -m 'not integration'

# Run a specific test file
uv run pytest tests/redteam/test_vulnerability_first.py -v

# Run integration tests (requires ORQ_API_KEY in .env)
uv run pytest -m integration

# Lint
uv run ruff check src

# Format
uv run ruff format src

# Type check — the whole repo. Never scope it to a path (see "Before pushing").
uv run basedpyright

# Build
uv build

# Serve the docs site locally (live-reload at http://127.0.0.1:8000/evaluatorq/)
uv run --group docs mkdocs serve

# Build the docs site (strict — fails on warnings, as CI does)
uv run --group docs mkdocs build --strict

# Validate mermaid diagrams render in strict renderers (GitHub/VS Code) — runs in CI.
# strict build does NOT catch mermaid label defects; this does.
uv run python scripts/validate_mermaid.py
```

When the user says **“do a test run”**, run the live trace validation for both
pipelines using the configured agent key:

```bash
ORQ_API_KEY=... EVALUATORQ_AGENT_KEY=... \
  uv run python scripts/live_trace_validation.py both
```

This runs 3 personas × 3 scenarios for agent simulation and a small hybrid red-team
check, then validates the root spans and run metadata. Use `orq traces list` to
inspect the resulting traces.

## Opening a PR

**Assign the PR to its author.** `gh pr create` leaves the assignee empty, so
open one with `--assignee @me` — an unassigned PR has no one the board can point
at when it stalls.

Reviewers need no flag: `.github/CODEOWNERS` requests them on every PR and skips
the author. Change that file, not the `gh` invocation, to change who reviews.

## Before pushing to a PR

Run the same checks CI runs, **verbatim**, before every push:

```bash
uv run ruff check src
uv run ruff format --check src
uv run basedpyright                 # whole repo — NOT a path
uv run pytest -m 'not integration'
```

**Do not scope `basedpyright` to a path.** CI runs it bare, which covers `tests/`
as well as `src/`. Running `uv run basedpyright src/` passes clean while CI fails
on type errors in test files — parametrized args annotated `str` where the
signature wants a `Literal`, raw dicts passed where a pydantic model is expected.
That exact mistake left PR #119 red across all four Python versions for three
commits without any local signal.

Note the asymmetry: **ruff** is scoped to `src` (tests are deliberately not
ruff-formatted, so `ruff format --check tests/` reports the whole tree as
unformatted — don't "fix" that). **basedpyright** is not scoped. Match CI, not
intuition.

CI does not run integration tests. Real-API coverage runs weekly via
`.github/workflows/examples-weekly.yml`, which opens an issue on failure rather
than blocking a PR.

## Package Map

```
src/evaluatorq/
├── evaluatorq.py, evaluators.py, pairwise*.py  # Core evaluation + pairwise entry points
├── contracts.py, types.py   # Cross-subpackage data models (RunManifest, LLMConfig, …)
├── cli.py                   # CLI entry point (evaluatorq / eq)
├── common/                  # SHARED MACHINERY — read the table below before writing anything here or near it
├── redteam/                 # eq redteam: adaptive/ (pipeline), backends/, frameworks/, reports/
├── simulation/              # eq simulate: runner/, agents/, generators/, reports/
├── dashboard/               # FastHTML dashboard (eq dashboard)
├── openresponses/           # OpenAI Responses API integration
├── tracing/                 # OTel setup + evaluation/run/job spans
└── integrations/            # LangChain, LangGraph, CrewAI, pydantic-ai, openai-agents
```

Read the directory itself for the file list — it is always current, this file is not.

## Need X? Use Y. Do not reinvent.

`common/` is the shared layer. Every module there exists because two surfaces
drifted apart and a review consolidated them. Adding a third copy is the failure
mode this table exists to prevent.

| Doing | Use | Never |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [orq-ai/evaluatorq](https://github.com/orq-ai/evaluatorq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
