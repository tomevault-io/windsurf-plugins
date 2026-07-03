---
trigger: always_on
description: This file provides guidance to coding agents (Claude Code, Codex) when
---

# AGENTS

This file provides guidance to coding agents (Claude Code, Codex) when
working with code in this repository. If you are not Claude Code (which
already reads parent directories), also check the parent directory for
`AGENTS.md`.

## Overview

`plasma-wiki` is a standalone plugin providing the **wiki** skill.

### Architecture

The `wiki/` Python package is organized into `cli/` (typer app), `core/`
(business logic), `skills/` (the plugin skill), `util/` (shared
utilities), and `_config/` (wiki config templates and the git merge
driver), with the pytest suite in `tests/`.

**Wiki** manages folder-based wikis with `_index.md` files, automatic
link generation, frontmatter management, and Obsidian integration.

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
```

The test suite uses `pytest` with `--doctest-modules` enabled.

## Consistency

The single most important pattern in this codebase is the pattern of
**adhering to patterns**. Every convention documented here exists so
that the code reads as if one person wrote it. This matters more than
any individual style preference because it enables:

- **Fast visual scanning** — when code follows predictable shapes,
  deviations jump out immediately
- **Regex-based refactoring** — consistent patterns mean
  find-and-replace works across the codebase
- **Trustworthy AI-generated code** — the user must be able to review
  the agent's output and have it look indistinguishable from their own

When writing or modifying code:

1. **Read the surrounding code first.** Match its patterns exactly —
   variable names, comment style, line breaking, method ordering,
   everything.
2. **Do not silently "improve" patterns.** If the existing code uses a
   particular structure, use that same structure in your current task.
   But if you see a genuinely better convention — clearer, safer, more
   idiomatic — **propose it explicitly.** The priority is consistency,
   not preservation of the status quo. Consistently good beats
   consistently bad, so make the case for why a change is worth the
   churn and the user will adopt it.
3. **Do not rename variables** that shadow outer scopes if it is
   sensible to reuse that variable name (and is unlikely to become a
   bug).
4. **Do not reformat** existing comments, reorder methods, or
   restructure working code unless specifically asked.
5. **Do not remove comments.** Line-by-line comments are intentional —
   they help the user maintain order and scan code quickly. Emulate
   existing comment patterns in new code.
6. **When in doubt, emulate.** Find the nearest analogous code in the
   codebase and mirror its structure.
7. **Preserve trailing newline patterns.** If a file ends with a
   trailing newline, keep it. If a file ends without one, don't add one.
   Match whatever the file already does.

### Adapting to the Codebase

The patterns documented here are a starting point, not an exhaustive
rulebook. The codebase is the authoritative style guide — these docs
just accelerate your ramp-up.

- **Pattern discovery over pattern memorization.** When working in a
  file, treat the local code as the authority. If a file uses a pattern
  not documented here, adopt it — don't introduce the documented pattern
  as a "correction."
- **Resolve conflicts in favor of local code.** If a documented pattern
  conflicts with what you see in the file you're editing, follow the
  file. Flag the discrepancy but don't "fix" it unilaterally.
- **New patterns propagate by observation.** The codebase evolves. When
  you encounter a pattern that's clearly intentional but not documented,
  follow it in your new code. The user will correct you if it's a
  mistake.
- **Scan before writing.** Before adding a new method, class, or module,
  find a few analogous examples in the codebase and mirror their
  structure. This applies to everything: error handling shape, docstring
  phrasing, test organization, import style, comment density.
- **Keep these docs up to date.** When you discover conventions or
  patterns through the user's feedback or codebase observation that
  aren't yet documented, add them to the appropriate `AGENTS.md`:
  repo-specific conventions belong in the repo's own file; org-wide
  conventions belong in the shared sections, which are maintained at the
  organization level and synced verbatim across repositories — make
  shared-section changes at the source (or flag them for promotion),
  never in a synced copy.

**Propose better conventions.** If you see a pattern that could be
improved across the codebase — a more readable structure, a safer error
handling approach, a cleaner naming convention — say so. Explain *why*
it's worth the migration cost. The user values consistency over any
particular style, and will always prefer being consistently good over
consistently familiar. The rule is: don't deviate silently, but do
advocate openly.

## Templates


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [plasma-ai/wiki](https://github.com/plasma-ai/wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
