---
trigger: always_on
description: - Use the project environment managed by `uv` (see `README.md`).
---

# Project Guidelines

## Build And Test

- Use the project environment managed by `uv` (see `README.md`).
- Preferred bootstrap: `./setup.sh`.
- Python requirement is `>=3.14.0` (see `pyproject.toml`).
- Standard full checks:
	- `uv run ruff check .`
	- `uv run ruff format --check .`
	- `uv run mypy axis`
	- `uv run pytest`
- After code changes, run targeted tests for touched files first; run broader validation when shared behavior is affected.

## Architecture

- Keep boundaries clear:
	- `axis/interfaces/` contains API handlers and transport-facing logic.
	- `axis/models/` contains request/response models, enums, and parsing helpers.
	- `axis/device.py` and `axis/interfaces/vapix.py` orchestrate device and handler lifecycle.
- Follow the phase-based handler initialization model documented in `README.md` (`API_DISCOVERY`, `PARAM_CGI_FALLBACK`, `APPLICATION`).
- Prefer boundary normalization for incoming values (for example, enum coercion and defaults in model constructors/post-init).

## Conventions
- For every new model or interface file, add a brief, informative module-level docstring at the top of the file. Summarize the file's purpose and, for VAPIX-related code, reference the relevant VAPIX API or feature. Use the official VAPIX library documentation as a source for the summary when possible.

- Prefer minimal, targeted changes that preserve existing behavior unless the task explicitly requires a behavior change.
- Do not modify unrelated code, formatting, or tests.
- Never revert user changes unless explicitly asked.
- Before changing patterns or APIs, inspect nearby code and follow existing local style.
- Prefer root-cause fixes over workarounds.
- In `axis/models/`, keep enum members and dataclass fields alphabetized to match local style when practical; for dataclasses, preserve Python's required-before-default field rule even when that prevents perfect alphabetical ordering.
- For enums and external inputs, preserve existing defensive normalization patterns (for example `_missing_` fallbacks and constructor normalization).
- For event/XML handling, preserve namespace-aware parsing and root-shape guards instead of assuming a fixed payload shape.
- For request models, keep the generic `ApiRequest[T]` contract aligned with runtime `response_type` metadata on every subclass.
- Use `BytesResponse` for write/raw-byte request classes instead of implicit defaults.

## Testing Conventions

- Add or update focused tests in the nearest relevant `tests/` module when behavior changes.
- Reuse existing async fixtures and HTTP mocking patterns from `tests/conftest.py`.
- If tests, typing, or linting fail for unrelated pre-existing reasons, report that clearly instead of fixing unrelated code.
- Expect commit hooks to run Ruff, Ruff format, and mypy; if hooks modify files, re-stage and re-run checks.

## Git Workflow

- Never create commits on the `master` branch.
- Never push commits directly to the `master` branch.
- Before any commit or push, check the current branch and confirm it is not `master`.
- If work is currently on `master`, create or switch to a feature branch before committing.
- If asked to commit or push from `master`, explain that the change must go through a feature branch and pull request.
- For any requested git operation, verify branch state first and summarize what will happen before committing or pushing.

## How to Ask Copilot for High-Quality Work

### Request Template

Use this template when asking Copilot to implement, fix, or refactor. Fill in each section to ensure clear scope and expectations:

```
[CONTEXT]
I'm working on [area: new handler / bug fix / test / refactor].
This touches [files/modules affected].
Related issue: [link or #number].

[CONSTRAINTS]
- Preserve existing behavior unless explicitly asked to change
- Follow the phase-based handler initialization model (see README.md)
- New code must have 100% test coverage
- Overall coverage must stay ≥95%
- Reuse fixtures from tests/conftest.py
- If enums are involved, include _missing_ fallback
- Every ApiRequest subclass must declare an explicit response_type consistent with its generic argument
- Keep changes minimal and targeted (no unrelated refactoring)

[TASK]
Please [implement / fix / refactor]:
1. [Specific step 1 with context]
2. [Specific step 2 with context]
3. [Specific step 3 with context]

[ACCEPTANCE CRITERIA]
- ✓ All checks pass: `uv run ruff check .`, `uv run ruff format --check .`, `uv run mypy axis`, `uv run pytest`
- ✓ Tests added in `tests/test_<area>.py` covering new code (100% coverage)
- ✓ No unrelated changes
- ✓ Follows architectural layer boundaries (models vs. interfaces)

[VERIFICATION COMMAND]
uv run ruff check .
uv run ruff format --check .
uv run mypy axis
uv run pytest tests/test_<area>.py -v --cov=axis.<area> --cov-report=term-missing
```

### Tips

- **For new handlers:** Mention the API endpoint and expected response shape.
- **For bug fixes:** Paste the error trace and describe expected behavior.
- **For tests:** Link to the code under test and describe missing scenarios.
- **For reviews:** Use the Axis Review or Axis Review Verify agents instead (see below).

---

## How to Request Copilot Code Review

### Code Review Template


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kane610/axis](https://github.com/Kane610/axis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
