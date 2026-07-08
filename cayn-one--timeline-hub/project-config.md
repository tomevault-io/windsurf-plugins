---
trigger: always_on
description: - Prefer the smallest clear change that solves the current problem.
---

# Repository Agent Rules

## General defaults

- Prefer the smallest clear change that solves the current problem.
- Preserve existing behavior unless the request explicitly changes it.
- Avoid speculative architecture for hypothetical scale.
- If output expectations are unclear, ask before producing the final artifact.
- In plan/spec mode, prefer numbered clarification questions over assumptions.
- Keep recommendations scoped and actionable.
- Distinguish between:
  - now — required due to active impact
  - later — optional until a real trigger appears
- Do not introduce new abstractions or helpers unless required to solve the task correctly.
- Do not refactor unrelated code.
- Do not modify unrelated code or documentation.
- When editing docs, preserve existing structure and tone unless the task asks for a rewrite.
- When the response format is unconstrained, after completing a task, propose a short, high-signal next step or action that the user is likely to want next.

## Tooling

Environment and workflow:
- Use `uv` for dependency management and command execution.
- Add dependency ranges in `pyproject.toml`; rely on `uv.lock` for exact resolved versions.
- Install the dev environment with `uv sync --dev`.
- The repository uses `src/`; tests must import the installed package.
- Do not modify `sys.path` in tests or use pytest/pythonpath hacks.

Testing:
- Run tests with `uv run pytest`.
- Tests marked `live` hit external services and are skipped by default.
- Run live tests with `uv run pytest --live` when changing live integration coverage or when verifying external-service behavior.
- Keep existing unittest-style tests working under pytest.
- New tests should use pytest style unless there is a specific reason not to.

Test quality:
- Prefer fewer high-signal tests over broad low-value coverage.
- Add tests only when they protect a real invariant, failure mode, data contract, or behavior that would hurt if it silently changed.
- Avoid tests that merely assert default argument values, dataclass field defaults, trivial constructor wiring, or constant policy choices. Test explicit input -> behavior instead.
- Test defaults directly only when the default is an intentional stable public contract with meaningful user-facing behavior.
- Avoid tests that depend on exact checked-in config values or repo policy thresholds unless those exact values are an intentional stable public contract. Prefer explicit test fixtures that assert parsing, override precedence, validation, or resulting behavior.
- Avoid `__repr__` / formatting tests unless the exact representation is a stable public contract consumed by humans/tools. Do not test arbitrary visual output.
- Avoid tests that simply mirror implementation details, private helper structure, call order, or current refactor shape without validating observable behavior.
- Avoid dummy smoke tests that only instantiate objects or assert that code “does not crash” unless the absence of crashing is itself a meaningful regression boundary.
- Prefer tests for durable invariants: validation rules, conversion semantics, ordering guarantees, failure handling, idempotency, boundary behavior, persistence format, and externally visible API behavior.
- For data/domain code, prioritize edge cases that would corrupt stored data, hide invalid provider behavior, break replayability, or silently change model-facing features.
- If a change is purely mechanical and no durable behavior is affected, it is acceptable to add no tests rather than create weak tests.

Pre-commit:
- Pre-commit is the final enforcement gate.
- Do not bypass hooks.
- If hooks fail, fix the issues before committing.
- Ruff and Pyright must pass through the normal workflow.
- After completing any coding task, run `uv run pre-commit run --all-files`.
- Do not stop after tests, Ruff, or Pyright alone if pre-commit has not been run.
- Treat a failing pre-commit run as unfinished work and fix the issues before handing the result back for review.

## Code style

- Target modern Python as defined by `pyproject.toml`.
- Prefer current language features over legacy compatibility patterns.
- Do not add backward-compatibility patterns unless explicitly required.
- Prefer explicit code over clever abstractions.
- Prefer public documented APIs over private internals.
- Prefer fixing root causes over `Any`, `cast`, broad suppressions, or fragile hacks.
- Prefer explicit `None` checks and proper type narrowing.
- Use single quotes for normal strings.
- Use triple double quotes for triple-quoted strings and docstrings.
- Use Google-style docstrings.
- Do not add `from __future__ import annotations` unless there is a real need.
- Prefer snake_case naming.
- Use absolute imports from top-level packages.
- Do not rely on private attributes or methods unless explicitly required or there is no viable public alternative.
- Document intentional contract-level exceptions in `Raises:` sections.
- Do not document incidental internal exceptions unless they are part of intended API behavior.
- In behavioral and API-facing modules, prefer top-down organization:
  - public constants, types, and protocols
  - public classes and functions
  - private methods
  - private helper functions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cayn-one/timeline-hub](https://github.com/cayn-one/timeline-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
