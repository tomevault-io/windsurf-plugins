---
trigger: always_on
description: This project is a Python library.
---

# Project Structure

This project is a Python library.

# Code Guidelines

## Code Style

- Comments should be used very rarely. Code should generally express its intent.
- Never write a one-line docstring — either the name is sufficient or the behavior warrants a full explanation.
- Don't sort or remove imports manually — pre-commit handles it.
- Always include type hints for pyright in Python
- Respect the pyright rule reportUnusedCallResult; assign unneeded return values to `_`
- Prefer keyword-only parameters (unless a very clear single-argument function): use `*` in Python signatures and destructured options objects in TypeScript.
- When disabling a linting rule with an inline directive, provide a comment at the end of the line (or on the line above for tools that don't allow extra text after an inline directive) describing the reasoning for disabling the rule.
- Avoid telling the type checker what a type is rather than letting it prove it. This includes type assertions (`as SomeType` in TypeScript, `cast()` in Python) and variable annotations that override inference. Prefer approaches that let the type checker verify the type itself: `isinstance`/`instanceof` narrowing, restructuring code so the correct type flows naturally, or using discriminated unions. When there is genuinely no alternative, add a comment explaining why the workaround is necessary and why it is safe.

## Testing

- Always run tests with an explicit path (e.g. uv run pytest tests/unit) — test runners discover all types (unit, integration, E2E...) by default.
- When iterating on a single test, run that test in isolation first and confirm it is in the expected state (red or green) before widening to the full suite. Use the most targeted invocation available: a specific test function for Python (e.g. `uv run pytest path/to/test.py::test_name --no-cov`) or a file path and name filter for TypeScript (e.g. `pnpm test-unit -- path/to/test.spec.ts -t "test name" --no-coverage`). Only run the full suite once the target test behaves as expected.
- Test coverage requirements are usually at 100%, so when running a subset of tests, always disable test coverage to avoid the test run failing for insufficient coverage.
- Avoid magic values in comparisons in tests in all languages (like ruff rule PLR2004 specifies)
- Prefer using random values in tests rather than arbitrary ones (e.g. the faker library, uuids, random.randint) when possible. For enums, pick randomly rather than hardcoding one value.
- Avoid loops in tests — assert each item explicitly so failures pinpoint the exact element. When verifying a condition across all items in a collection, collect the violations into a list and assert it's empty (e.g., assert [x for x in items if bad_condition(x)] == []).
- When asserting a mock or spy was called with specific arguments, always constrain as tightly as possible. In order of preference: (1) assert called exactly once with those args (`assert_called_once_with` in Python, `toHaveBeenCalledExactlyOnceWith` in Vitest/Jest); (2) if multiple calls are expected, assert the total call count and use a positional or last-call assertion (`nthCalledWith`, `lastCalledWith` / `assert_has_calls` with `call_args_list[n]`); (3) plain "called with at any point" (`toHaveBeenCalledWith`, `assert_called_with`) is a last resort only when neither the call count nor the call order can reasonably be constrained.

### Python Testing

- When using `mocker.spy` on a class-level method (including inherited ones), the spy records the unbound call, so assertions need `ANY` as the first argument to match self: `spy.assert_called_once_with(ANY, expected_arg)`
- Before writing new mock/spy helpers, check the `tests/unit/` folder for pre-built helpers in files like `fixtures.py` or `*mocks.py`
- When a test needs a fixture only for its side effects (not its return value), use `@pytest.mark.usefixtures(fixture_name.__name__)` instead of adding an unused parameter with a noqa comment
- Use `__name__` instead of string literals when referencing functions/methods (e.g., `mocker.patch.object(MyClass, MyClass.method.__name__)`, `pytest.mark.usefixtures(my_fixture.__name__)`). This enables IDE refactoring tools to catch renames.
- When using the faker library, prefer the pytest fixture (provided by the faker library) over instantiating instances of Faker.
- **Choosing between cassettes and mocks:** At the layer that directly wraps an external API or service, strongly prefer VCR cassette-recorded interactions (via pytest-recording/vcrpy) — they capture real HTTP traffic and verify the wire format, catching integration issues that mocks would miss. At layers above that (e.g. business logic, route handlers), mock the wrapper layer instead (e.g. `mocker.patch.object(ThresholdsRepository, ...)`) — there is no value in re-testing the HTTP interaction from higher up.
- **Never hand-write VCR cassette YAML files.** Cassettes must be recorded from real HTTP interactions by running the test once with `--record-mode=once` against a live external service: `uv run pytest --record-mode=once <test path> --no-cov`. The default mode is `none` — a missing cassette will cause an error, which is expected until recorded.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LabAutomationAndScreening/pyalab](https://github.com/LabAutomationAndScreening/pyalab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
