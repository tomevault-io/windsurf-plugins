---
trigger: always_on
description: This file provides guidance to coding agents (Claude Code, Codex) when working
---

# AGENTS

This file provides guidance to coding agents (Claude Code, Codex) when working
with code in this repository. If you are not Claude Code (which already reads
parent directories), also check the parent directory for `AGENTS.md`.

Especially important points to keep in mind (each expanded in its section
below):

- **Questions are not edit requests.** When a message asks a question without
  explicitly requesting a change, answer and stop (Communication).
- **Write clear and concise English.** Write clear, concise, plain English which
  can be quickly and easily parsed by the user - answer questions directly in
  short form unless elaboration is requested (Writing Style).
- **Seek the best solution, not agreement.** When you think the user is wrong,
  don't be afraid to say so — the right answer matters more than the path of
  least resistance (Pushing Back).

## Overview

`plasma-fractal` is a standalone plugin providing the **fractal** skill.

The `fractal/` package is organized into `cli/` (typer app), `core/` (business
logic), `tui/` (Textual app), `impl/` (provider backends), `skills/` (the plugin
skill), `util/` (shared utilities), and the node machinery seeds — `_assets/`,
`_node/`, and `_scripts/` — with the pytest suite in `tests/`.

`shim/` holds the metadata-only `fractal` pointer dist for PyPI: no code, just
an exact `plasma-fractal==<version>` pin that bumps in lockstep with every
release surface. The build workflow gates and builds it alongside the main
package, and the publish job uploads both dists — both PyPI projects must trust
that workflow (same repository, workflow file, and environment) as their
publisher.

`Node` delegates lifecycle operations to the `_scripts/` shell scripts via
`subprocess.run()`. The iteration loop is in-process Python: `start.sh` execs
`fractal node _loop` inside the tmux session, and `Loop`
(`fractal/core/loop.py`) drives each iteration end to end — prompt assembly,
agent launches, run/iter/step row accounting through `node.record`, and the
work-product commit pipeline in `fractal/core/commit.py`.

Agent invocation is a core/impl seam: `fractal/core/agent.py` defines the
`Agent` base class (invocation argv, stream parsing, cost and logging hooks)
plus the name registry (`resolve`/`register`/`supported`), and
`fractal/impl/{claude,codex,grok,opencode,omp}.py` are the provider backends — a
new provider slots in as one new `impl/` module registered in `core/agent.py`.

## Build & Development

```bash
# install dev dependencies (creates a .venv if none is active)
./install.sh --all-extras --groups=test,lint,type

# or set up the environment manually
uv sync --all-extras --group test --group lint --group type
uv run pre-commit install

# run tests
uv run --no-sync pytest

# run pre-commit
uv run --no-sync pre-commit run [--all-files]

# run type check (not enforced)
uv run --no-sync pyright

# run security scan (not enforced)
uv sync --inexact --group security
uv run --no-sync safety scan
```

The test suite uses `pytest` with `--doctest-modules` enabled. Integration tests
create real git repos and worktrees — session-scoped to avoid repeated
`fractal node init` overhead.

## Consistency

The single most important pattern in this codebase is the pattern of **adhering
to patterns**. Every convention documented here exists so that the code reads as
if one person wrote it. This matters more than any individual style preference
because it enables:

- **Fast visual scanning** — when code follows predictable shapes, deviations
  jump out immediately
- **Regex-based refactoring** — consistent patterns mean find-and-replace works
  across the codebase
- **Trustworthy AI-generated code** — the user must be able to review the
  agent's output and have it look indistinguishable from their own

When writing or modifying code:

1. **Read the surrounding code first.** Match its patterns exactly — variable
   names, comment style, line breaking, method ordering, everything.
2. **Do not silently "improve" patterns.** If the existing code uses a
   particular structure, use that same structure in your current task. But if
   you see a genuinely better convention — clearer, safer, more idiomatic —
   **propose it explicitly.** The priority is consistency, not preservation of
   the status quo. Consistently good beats consistently bad, so make the case
   for why a change is worth the churn and the user will adopt it.
3. **Do not rename variables** that shadow outer scopes if it is sensible to
   reuse that variable name (and is unlikely to become a bug).
4. **Do not reformat** existing comments, reorder methods, or restructure
   working code unless specifically asked.
5. **Do not remove comments.** Line-by-line comments are intentional — they help
   the user maintain order and scan code quickly. Emulate existing comment
   patterns in new code.
6. **When in doubt, emulate.** Find the nearest analogous code in the codebase
   and mirror its structure.
7. **End files with a trailing newline.** Every committed file ends with one —
   the `end-of-file-fixer` hook enforces it.

### Adapting to the Codebase

The patterns documented here are a starting point, not an exhaustive rulebook.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [plasma-ai/fractal](https://github.com/plasma-ai/fractal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
