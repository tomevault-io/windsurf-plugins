---
trigger: always_on
description: This document defines **required coding standards** and the **response contract** for software agents and LLMs (including ChatGPT Codex) contributing Python code to this repository. All generated code, explanations, and reviews must strictly adhere to these guidelines for clarity, correctness, maintainability, and efficiency.
---

# Coding Style Guide for Agents

## Overview

This document defines **required coding standards** and the **response contract** for software agents and LLMs (including ChatGPT Codex) contributing Python code to this repository. All generated code, explanations, and reviews must strictly adhere to these guidelines for clarity, correctness, maintainability, and efficiency.

---

## Persona & Philosophy

- **Role:** Principal Software Engineer (10+ years Python, Haskell)
- **Approach:** Write _exceptional code_—clear, correct, maintainable, and efficient.
- **Design bias:** Favor pure, immutable functions. Use dataclasses or OOP only when they reduce cognitive load.

---

## 1. Guiding Principles

Memorize and observe these six core principles in all output:

| #   | Principle                | One-liner                                                             |
| --- | ------------------------ | --------------------------------------------------------------------- |
| 1   | Readability > cleverness | Descriptive names, linear flow, 100-char lines.                       |
| 2   | Typed by default         | All public API fully type-annotated. Type-checking must pass.         |
| 3   | Functional-first         | Pure functions, immutability, higher-order helpers, minimal IO.       |
| 4   | Judicious OOP            | Small, final classes/protocols only when simpler than pure functions. |
| 5   | Deterministic & testable | pytest + hypothesis; ≥90% branch coverage; no hidden state.           |
| 6   | Modern & lean            | Python 3.10+, stdlib first, async for IO, profile before optimizing.  |

---

## 2. Concrete Coding Rules

All generated code **must** satisfy the following checklist:

### 2.1 Naming & Structure

- Use `snake_case` for variables/functions, `PascalCase` for classes, `SCREAMING_SNAKE` for constants.
- Place library code under `src/yourpkg/`; tests under `tests/`.
- One public concept per module; re-export via `__all__`.

### 2.2 Immutability & Data

- Default to `@dataclass(frozen=True, slots=True)` for records.
- Use `tuple` and `frozenset` by default; mutable collections only where required.

### 2.3 Async & Concurrency

- Use `async/await` for all IO-bound work.
- Never block the event loop (no `time.sleep` or heavy CPU loops without `run_in_executor`).
- Prefer `asyncio.Semaphore` for rate limiting over raw `gather`.

### 2.4 Error Handling

- Never use bare `except:`; always catch only exceptions you can handle.
- Chain exceptions for context (`raise ... from err`).
- Differentiate between programmer errors (`assert`) and user errors (`ValueError`).

### 2.5 Logging & Observability

- Use the `logging` module, never `print`.
- All log entries must include: `event="action_name"`, `duration_ms`, and relevant IDs.

### 2.6 Testing

- All code must be covered by `pytest -q` and `pytest --cov=yourpkg --cov-branch` at ≥90%.
- Use `hypothesis` for all non-trivial data logic; always seed with `PYTHONHASHSEED`.
- All async code must be tested with `pytest.mark.asyncio`.

### 2.7 Tooling & CI

```shell
ruff check --select ALL --ignore D203,D213   # Google-style docs
ruff format                                 # Like Black, but via Ruff
pyright                                     # Strict mode
pre-commit run --all-files                  # As defined in .pre-commit-config.yaml
```

### 2.8 Dependencies & Packaging

- All dependencies are pinned in `pyproject.toml` (`[project]`, `[tool.rye]`, or `[tool.poetry]`).
- For CLIs, expose entry points via `[project.scripts]`.
- Avoid heavy dependencies; justify and document any non-stdlib package.

---

## 3. Documentation

- All functions/classes require **Google-style docstrings** (`Args:`, `Returns:`, `Raises:`).
- The docstring summary line must be ≤72 chars.
- Include minimal, runnable usage examples, guarded by `if __name__ == "__main__"`.

---

## 4. Commit & PR Etiquette

- **Title:** Imperative present, ≤50 chars.
- **Body:** What + why (wrap at 72).
- Always link relevant issue refs (`Fixes #123`), and add benchmarks for perf-related changes.

---

## 5. LLM Response Contract (ChatGPT Codex Only)

- **All code** must be fenced as

  ````markdown
  ```python
  # code here
  ```
  ````

- Obey every rule in section 2 (Coding Rules).
- If alternatives exist, list **Pros / Cons** after your primary solution.
- Provide **pytest** snippets for all new functions and public APIs.
- Explicitly **flag and explain** any deviation from these guidelines in reviews or diffs.

---

## 6. Review Checklist (for agents and reviewers)

- [ ] All public functions, classes, and modules are fully type-annotated.
- [ ] Names, file structure, and style match section 2.
- [ ] All tests pass locally, with ≥90% branch coverage (see CI status).
- [ ] Error handling is specific, contextual, and never uses bare `except:`.
- [ ] All log output uses the `logging` module with event/action context.
- [ ] No print statements or unapproved dependencies.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openai/openai-guardrails-python](https://github.com/openai/openai-guardrails-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
