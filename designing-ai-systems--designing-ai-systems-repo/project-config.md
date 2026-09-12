---
trigger: always_on
description: > Derived from Simon Willison's [Agentic Engineering Patterns](https://simonwillison.net/guides/agentic-engineering-patterns/) and Andrej Karpathy's [LLM coding pitfalls](https://github.com/forrestchang/andrej-karpathy-skills). These rules apply to every task in this repository.
---

# Cursor Rules — Agentic Engineering Patterns

> Derived from Simon Willison's [Agentic Engineering Patterns](https://simonwillison.net/guides/agentic-engineering-patterns/) and Andrej Karpathy's [LLM coding pitfalls](https://github.com/forrestchang/andrej-karpathy-skills). These rules apply to every task in this repository.

---

## 1. Think Before Coding

Do not assume. Do not hide confusion. Surface tradeoffs.

- **State assumptions explicitly.** If you are uncertain about the user's intent, ask before writing code. Never silently pick one interpretation and run with it.
- **Present multiple interpretations** when genuine ambiguity exists. Lay out the options with tradeoffs and let the user decide.
- **Push back when warranted.** If a simpler approach exists, say so. If the request will create unnecessary complexity, flag it.
- **Stop when confused.** Name exactly what is unclear and ask a targeted clarifying question. Do not produce speculative code to "figure it out."

---

## 2. Simplicity First

Write the minimum code that solves the problem. Nothing speculative.

- No features beyond what was asked.
- No abstractions for single-use code.
- No premature "flexibility" or "configurability" that was not requested.
- No error handling for impossible scenarios.
- If 200 lines could be 50, rewrite it as 50.
- Prefer boring, well-understood technology over novel approaches unless the user specifies otherwise.

**Self-check:** Would a senior engineer say this is overcomplicated? If yes, simplify.

---

## 3. Surgical Changes

Touch only what you must. Clean up only your own mess.

When editing existing code:
- Do NOT "improve" adjacent code, comments, or formatting that is unrelated to the task.
- Do NOT refactor things that are not broken.
- Match the existing code style, even if you would write it differently in a greenfield project.
- If you notice unrelated dead code or issues, mention them in your response — do NOT silently delete or fix them.

When your changes create orphans:
- Remove imports, variables, and functions that YOUR changes made unused.
- Do NOT remove pre-existing dead code unless explicitly asked.

**Self-check:** Every changed line must trace directly to the user's request. If it does not, revert it.

---

## 4. Red/Green Test-Driven Development

Every implementation MUST follow strict red/green TDD. No exceptions.

### The Protocol

1. **Write tests FIRST** that describe the desired behavior.
2. **Run the tests and confirm they FAIL** (the RED phase). If they pass already, the tests are not exercising new behavior — rewrite them.
3. **Implement the minimum code** to make the failing tests pass (the GREEN phase).
4. **Run all tests** to confirm nothing is broken.
5. **Refactor** only after all tests pass, and re-run tests after refactoring.

### What This Means in Practice

- "Add validation" becomes "Write tests for invalid inputs, then make them pass."
- "Fix the bug" becomes "Write a test that reproduces the bug, then make it pass."
- "Refactor X" becomes "Ensure all tests pass before AND after the refactor."
- Never write implementation code without a failing test that demands it.

### Test Quality Standards

- Tests must be independent and deterministic (no test ordering dependencies, no flaky timing).
- Test names must describe the behavior under test, not the implementation.
- Each test should verify one logical assertion or closely related group of assertions.
- Include edge cases: empty inputs, boundary values, error conditions, unicode, large payloads.
- Prefer integration tests for API endpoints and unit tests for pure logic.

---

## 5. First Run the Tests

At the start of every session against an existing codebase:

1. **Run the full test suite first** before making any changes. This gives you baseline knowledge of the project's health, its size, and its test patterns.
2. **Use the test suite to learn the codebase.** Tests are executable documentation; read them to understand expected behavior.
3. **After every change, run the full test suite** to confirm no regressions.

If the project uses Python: `uv run pytest` or `pytest`.
If the project uses Node: `npm test` or `npx jest`.
If unsure, look at `package.json`, `pyproject.toml`, `Makefile`, or CI config to discover the test command.

---

## 6. Agentic Manual Testing

Automated tests are necessary but not sufficient. After tests pass, manually verify the feature works end-to-end.

- For Python libraries: use `python -c "..."` to exercise functions with realistic and edge-case inputs.
- For CLI tools: run the tool with representative arguments and inspect output.
- For web APIs: start a dev server and probe endpoints with `curl`.
- For web UIs: use Playwright or a browser automation tool to verify the interface renders and behaves correctly.
- Write demo scripts in `/tmp` to avoid accidentally committing throwaway test files.

If manual testing uncovers an issue, fix it using red/green TDD so the case is permanently covered by automated tests.

---

## 7. Goal-Driven Execution


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [designing-ai-systems/designing_ai_systems_repo](https://github.com/designing-ai-systems/designing_ai_systems_repo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
