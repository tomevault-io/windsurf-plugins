---
trigger: always_on
description: Please refer to `README.md`, `docs/PLAN-separation.md`, `docs/IMPLEMENTATION.md`, and `docs/DEPLOY.md` for project-specific details.
---

# Reality Check - Development Guide

Please refer to `README.md`, `docs/PLAN-separation.md`, `docs/IMPLEMENTATION.md`, and `docs/DEPLOY.md` for project-specific details.
This `AGENTS.md`/`CLAUDE.md` is specifically for ground rules, process, and behavior notes.

## Shared Repo / Multi-Agent Safety (MUST FOLLOW)

This repo may be edited by multiple agents concurrently. Treat the working tree as shared state.

### Before any change (required)
- Run `git status --porcelain`.
- If the working tree has unrelated changes, proceed normally but do not touch them; stage only your own files (`git add <paths>` / `git add -p`).
- If you need to edit a file that already has changes you did not create (potential conflict), flag it and ask the user how to proceed.
- If there are merge conflicts, STOP and ask the user how to proceed.

### Never discard others’ work (hard rule)
- NEVER run any command that can delete/overwrite existing work unless the user explicitly instructs it.
  This includes (but is not limited to):
  - `git restore ...`
  - `git checkout -- ...`
  - `git checkout .`
  - `git reset --hard ...`
  - `git clean -fd ...`
  - `rm -rf ...` / overwriting redirects like `> file`
  - bulk rewrites that destroy local edits (e.g., aggressive formatters) unless requested

### Scope discipline
- Only edit files needed for the user’s current request.
- Do not “clean up”, refactor, or revert unrelated diffs in touched files.
- If you need to avoid committing unrelated diffs, use `git add <paths>` or `git add -p`; never “fix” by reverting other hunks.

### If conflicts are unavoidable
- Coordinate: propose a plan that preserves both sets of edits (e.g., separate commits/branches/patches),
  and wait for explicit user instruction before any destructive resolution.

## Project Overview

Reality Check is a framework for rigorous, systematic analysis of claims, sources, predictions, and argument chains. It provides:
- LanceDB-backed storage with semantic search
- Structured methodology for claim extraction and evaluation
- Evidence hierarchy and prediction tracking
- Claude Code plugin for workflow automation

## Key Directories

```
realitycheck/
├── scripts/          # Core Python CLI tools (db.py, validate.py, export.py, migrate.py)
├── tests/            # pytest test suite
├── integrations/     # Tool-specific integrations
│   ├── claude/       # Claude Code plugin and skills
│   │   ├── plugin/   # Plugin (commands/, hooks/, scripts/)
│   │   └── skills/   # Global skills (alternative to plugin)
│   └── codex/        # OpenAI Codex skills
├── methodology/      # Analysis methodology docs (extracted from framework)
├── docs/             # Development docs (PLAN-*.md, IMPLEMENTATION.md)
└── examples/         # Minimal example data
```

## Roles (Planners / Coders / Reviewers)

We use separate lanes for development work:

- **Planner**: produces/updates specs and punchlists in `docs/` (typically between sprints).
- **Coder**: owns all implementation patches (code + tests) and repo changes.
- **Reviewer**: analysis-only; must not author implementation patches (no code changes).
- **Human lead**: arbitrates scope, risk, and disagreements; decides what is a blocker vs a deferral.

Rules:
- Reviewers provide findings + rationale + suggested fixes, but do not change the repo.
- Coders translate reviewer findings into tracked punchlist/checklist entries before implementing fixes.
- Reviewer follow-up is confirmation-only (resolved / unresolved with rationale), not code changes.

## Development Philosophy: Spec → Plan → Test → Implement

**This project is strictly spec/plan/test-driven.** The goal is a high-quality, maintainable framework that's easy to update and extend. Every feature follows this cycle:

### The Cycle

1. **Spec**: Define requirements clearly in `docs/` before writing any code
2. **Plan**: Create implementation plan with affected files (tree diagram)
3. **Test**: Write unit tests AND e2e tests BEFORE implementation
4. **Implement**: Write minimal code to pass the tests
5. **Validate**: Run full test suite - all tests must pass
6. **Commit**: Atomic commits with passing tests only

### Why Tests First?

- **Clarifies requirements** - Writing tests forces you to think through edge cases
- **Prevents scope creep** - You only implement what the tests require
- **Enables refactoring** - Tests catch regressions when you improve code later
- **Documents behavior** - Tests are executable specifications
- **Catches bugs early** - Faster to fix issues before code is "done"

### Test Coverage Requirements

- **Unit tests**: Every public function in `scripts/*.py` must have tests
- **E2E tests**: Every user workflow must have integration tests
- **Edge cases**: Error paths, empty inputs, boundary conditions
- **No skipping**: If a test fails, fix it before proceeding

## Documentation as Source of Truth

- Treat `docs/` as the source of truth and always prioritize keeping them up to date
- `docs/PLAN-separation.md` - Architecture and implementation plan
- `docs/IMPLEMENTATION.md` - Progress tracking (punchlist + worklog)
- `docs/DEPLOY.md` - Release quick reference
- `docs/PUBLISH.md` - Full release punch list (PyPI + GitHub)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lhl/realitycheck](https://github.com/lhl/realitycheck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
