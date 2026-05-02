---
trigger: always_on
description: Project-level guidance for coding agents working in this repository.
---

# AGENTS.md

Project-level guidance for coding agents working in this repository.

## Workflow Rules

- Never commit directly to `main`.
- If the current branch is `main`, create and switch to a new feature branch before making changes.
- Always work on a feature branch.
- Before the first commit on a newly created branch, if version files are touched, verify whether the version bump is major, minor, or patch and apply that decision consistently.
- Commit only files touched for the requested task.
- Do not include unrelated local artifacts in commits.
- Keep commits single-purpose (for example: `fix`, `refactor`, `test`, `docs`) and avoid mixing unrelated categories.

## Required Validation After Every Code Change

Run both checks after every code change, using the project config in `pyproject.toml`:

- `py -3.13 -m mypy --config-file pyproject.toml`
- `py -3.13 -m pytest -c pyproject.toml`

Do not report completion if either command fails.

## Interpreter And Test Execution

- Use the same Python interpreter pattern as the project commands (`py -3.13`) when running tests and type checks.
- Prefer explicit config-driven invocations over implicit defaults so local/global tooling differences do not affect results.
- If you run targeted tests while iterating, still run the full required validation commands before completion.

## Testing Conventions

- Put tests in the matching module area:
  - core/domain logic -> `tests/core`
  - UI behavior -> `tests/ui`
- Reuse shared test utilities/helpers as much as possible before adding new test scaffolding.
- Shared test utility locations in this repo:
  - Core tests: `tests/core/helpers.py` (payload/model builders like `make_valid_data`, `make_portfolio`)
  - UI tests: `tests/ui/conftest.py` (shared fixtures/builders like `qapp`, `make_plan_step`, `make_buy_calculation`)
- Always accompany new code with new tests, and update existing tests when behavior/contracts change.

## Documentation Maintenance

- If any file is added or deleted, update `docs/ARCHITECTURE.md` in the same task to reflect the structural change.
- If user-facing behavior changes, update `README.md` in the same task.
- Keep `README.md` user-facing; put implementation details in `docs/ARCHITECTURE.md`.
- In `README.md`, describe user outcomes and avoid implementation-specific details unless explicitly requested (for example: internal state flags, cache tiers, thread/worker lifecycle, or exact dialog button labels).
- For `README.md` updates, prefer happy-path user outcomes; implementation details, edge-case handling, and internal lifecycle notes belong in architecture docs or code comments.
- Prefer documenting behavior contracts over implementation details so docs remain stable across refactors.
- Keep documentation concise and behavior-accurate; avoid broad background text when a precise statement is enough.
- Be strict with absolutes in user-facing docs: verify `always` / `never` / `only` claims against current runtime paths before finishing.
- Keep architecture/docs granularity symmetric: avoid over-documenting one field/module unless peers are documented at the same level or the exception is intentional.
- When changing user-facing text (UI labels/messages/docs), verify wording matches runtime behavior and units.

## Review Quality Rules

- For branch reviews, review changes relative to the branch merge-base with `main`.
- Use `main...HEAD` (merge-base diff), not latest-commit diffs, for branch-level review scope.
- After fixing review findings, do a quick follow-up self-review of touched areas before handing off.
- For documentation-only changes, verify claims map to current code paths and avoid redundant repetition.

## Refactor And Consistency Rules

- Prefer enum-driven behavior over hardcoded member checks (for example, use `exchange.currency` instead of branching on specific exchange names).
- Use shared defaults/constants for fallbacks (for example, `DEFAULT_EXCHANGE`) instead of inline string literals.
- When a component's responsibility expands, rename stale narrow identifiers in the same task when practical (types, methods, constants, and related docs/comments).
- For schema or field renames, update all affected layers in the same task: model, JSON IO, validation, UI labels/tooltips, tests, and docs.
- After intentional breaking changes, remove stale "legacy" compatibility wording unless backward compatibility is still implemented.
- When introducing a canonical public API, migrate callers in the same task and remove or privatize legacy setters unless an explicit migration window is required.
- Remove stale no-op seams once behavior has stabilized; do not keep indefinite compatibility wrappers with no runtime effect.
- When removing a seam/wrapper, also remove stale protocol members and seam-specific tests in the same task.
- In PowerShell commands, avoid Bash-only chaining like `&&`; use separate commands or `;`.

## PR Writing

- When asked for a PR title and description, provide a short description written in Markdown.

## Skills

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [urikirsh/investment-planner](https://github.com/urikirsh/investment-planner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
