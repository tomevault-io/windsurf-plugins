---
trigger: always_on
description: Always run Python commands via `uv run`, e.g.:
---

# Claude Code instructions for orcapod-python

## Running commands

Always run Python commands via `uv run`, e.g.:

```
uv run pytest tests/
uv run python -c "..."
```

Never use `python`, `pytest`, or `python3` directly.

## Updating agent instructions

When adding or changing any instruction, update BOTH:
- `CLAUDE.md` (for Claude Code)
- `.zed/rules` (for Zed AI)

## Design issues log

`DESIGN_ISSUES.md` at the project root is the canonical log of known design problems, bugs, and
code quality issues.

When fixing a bug or addressing a design problem:
1. Check `DESIGN_ISSUES.md` first — if a matching issue exists, update its status to
   `in progress` while working and `resolved` once done, adding a brief **Fix:** note.
2. If no matching issue exists, ask the user whether it should be added before proceeding.
   If yes, add it (status `open` or `in progress` as appropriate).

When discovering a new issue that won't be fixed immediately, ask the user whether it should be
logged in `DESIGN_ISSUES.md` before adding it.

## Superpowers artifacts

Place all superpowers-related artifacts (design specs, plans, etc.) in the `superpowers/`
directory at the project root — **not** under `docs/`. The `docs/` directory is reserved for
actual library documentation.

- Specs go in `superpowers/specs/`

## Backward compatibility

This is a greenfield project pre-v0.1.0. Do **not** add backward-compatibility shims,
re-exports, aliases, or deprecation wrappers when making design or implementation changes.
Just change the code and update all references directly.

## No `sys.modules` hacks

Never manipulate `sys.modules` directly (e.g. `sys.modules.setdefault`). If a subpackage
import path doesn't work, create a proper re-export package with an `__init__.py` instead.

## Docstrings

Use [Google style](https://google.github.io/styleguide/pyguide.html#38-comments-and-docstrings)
Python docstrings everywhere.

## Linear issue tracking

All work must be linked to a Linear issue. Before starting any feature, bug fix, or
refactor:

1. **Check for an existing issue** — search Linear for a corresponding issue.
2. **If none exists** — ask the developer whether to create one. Do not proceed without
   either a linked issue or explicit approval to skip.
3. **When starting work on an issue** — update its Linear status to **In Progress**.
4. **When a new issue is discovered** during development (bug, design problem, deferred
   work), create a corresponding Linear issue using the template below.

When creating Linear issues, always use this template for the description:

```
## Overview
What is this project about? Describe the problem space and the high-level approach.

## Goals & Success Criteria
* Specific, measurable outcomes.

## Scope & Boundaries
(Optional — remove if not needed.)
In scope:
* ...
Out of scope:
* ...

## Dependencies & Risks
(Optional — remove if none.)
* ...

## Resources & References
(Optional — remove if none.)
* ...

## Milestones
(Optional — only for projects longer than ~4 weeks. Remove for shorter projects.)
* ...
```

Remove any optional sections that don't apply rather than leaving them empty.

### Branches and PRs

When working on a feature, create and checkout a git branch using the `gitBranchName`
returned by the primary Linear issue (e.g. `eywalker/plt-911-add-documentation-for-orcapod-python`).

**Feature branch PRs always target the `dev` branch.** The `dev` → `main` PR is used
for versioning/releases only.

If a feature branch / PR corresponds to multiple Linear issues, list all of them in the
PR description body so that Linear's GitHub integration auto-tracks the PR against each
issue. Use the format `Fixes PLT-123` or `Closes PLT-123` (GitHub magic words) for issues
that the PR fully resolves, and simply mention `PLT-456` for issues that are related but
not fully resolved by the PR.

## Responding to PR reviews

When asked to respond to PR reviewer comments:

1. **Read all comments carefully** — fetch every review comment on the PR before forming any opinion.
2. **Evaluate each comment** — decide whether to accept, partially accept, or decline, and why.
3. **Present a revision plan** — show the full plan (table: comment summary, verdict, proposed action)
   and **wait for user approval before touching any code or posting any replies**.
4. **Fix, then reply** — once approved, make all fixes in a single commit, then post replies to each
   reviewer comment explaining what was done (or why it was declined).

Never implement changes or reply to reviewers before the plan has been approved.

## Git commits

Always use [Conventional Commits](https://www.conventionalcommits.org/) style:

```
<type>(<optional scope>): <short description>
```

Common types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `perf`, `ci`.

Examples:
- `feat(schema): add optional_fields to Schema`
- `fix(packet_function): reject variadic parameters at construction`
- `test(function_pod): add schema validation tests`
- `refactor(schema_utils): use Schema.optional_fields directly`

---

## Project layout

```
src/orcapod/
├── types.py                    # Schema, ColumnConfig, ContentHash
├── system_constants.py         # Column prefixes and separators

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nauticalab) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
