---
trigger: always_on
description: What to test — never test library functionality, never hardcode Typed defaults, documentation rules pointer.
---

# What to Test

- Write comprehensive test coverage for all features
- Update tests immediately when making breaking changes
- Test both success and failure cases (use `pytest.raises` for expected exceptions)
- Include edge cases and error conditions (empty inputs, `None` values, boundary values, concurrent access)
- Use descriptive test names: `test_worker_stop_during_active_submission_cancels_pending_futures`
- Follow the AAA pattern: Arrange (setup), Act (call), Assert (verify)
- Use `@pytest.mark.parametrize` to cover multiple scenarios from a single test function
- Never catch and suppress test failures — fix the code or the test

## Never Test Imported Library Functionality (CRITICAL — LLM Anti-Pattern)

**Tests must target YOUR code's behavior, not the behavior of imported libraries.** If a function works because a library (morphic, Pydantic, Ray, etc.) does its job correctly, you do not need a test that verifies the library works. The library's own test suite covers that. Your test suite covers YOUR logic.

**Why LLMs produce this anti-pattern:** LLM coding assistants reflexively generate "smoke tests" that verify framework behavior rather than application behavior. When asked to "add tests for `@validate`," an LLM will write tests that pass a wrong type and assert `ValidationError` is raised. This tests that `morphic.validate` works — it does not test that your function computes the correct result, handles edge cases, or integrates correctly with the rest of the system. These tests provide zero value: they will never fail unless someone upgrades morphic to a broken version, which is not your test suite's job to catch.

**The decision rule:** before writing a test, ask: **"If this test fails, does it mean MY code has a bug, or does it mean a LIBRARY has a bug?"** If the answer is "a library has a bug," do not write the test.

| ❌ Tests that target library behavior (NEVER write these) | ✅ Tests that target YOUR behavior (ALWAYS write these) |
|---|---|
| `@validate` rejects wrong types with `ValidationError` | `wait()` returns done and not-done sets with correct futures in each |
| `Typed` fields are immutable after construction | `gather()` preserves input order when given a list of futures |
| `Registry.of("name")` resolves the correct subclass | `WorkerBuilder` raises `ValueError` for incompatible mode + limits combo |
| `model_dump()` serializes all fields | Worker pool distributes tasks across workers via load balancer |
| `@field_validator` runs during construction | `RetryConfig` retry loop stops after `max_retries` exhausted |
| Pydantic coerces `"25"` to `int(25)` | `LimitSet.acquire()` blocks when capacity is exhausted |
| `ray.wait()` returns ready and not-ready refs | `stop()` cancels pending futures when called during active submission |

**Specific libraries this applies to in this codebase:**
- `morphic.validate`, `morphic.Typed`, `morphic.Registry`, `morphic.MutableTyped` — assume they work
- `pydantic.ValidationError`, `@field_validator`, `model_dump()`, `PrivateAttr` — assume they work
- `ray`, `ray.wait()`, `ray.get()`, `ray.remote` — assume they work
- `concurrent.futures`, `threading`, `multiprocessing` — assume they work
- `pytest.raises`, `pytest.mark.parametrize`, pytest fixtures — assume they work

**What you SHOULD test:** the logic YOUR code adds on top of these libraries. If `gather()` polls futures with a `PollingStrategy` and returns results in input order, test that the ordering is preserved and the polling terminates — not that `ray.wait()` returns the correct ObjectRefs or that `@validate` rejects a dict where a `List[Future]` is expected.

## Never Hardcode Typed Default Values in Tests (CRITICAL)

**When testing a method that receives parameters from a `Typed` class's fields, use `ClassName.param_default_values` to supply defaults — never re-type the values manually.** `param_default_values` is a `@classproperty` on every Morphic `Typed` class that returns a `Dict[str, Any]` of all fields that have defaults, mapped to their default values.

**Why this matters:**

1. **Silent divergence.** When a class field default changes (e.g., `GlobalDefaults.worker_timeout` moves from `30.0` to `60.0`), every test that hardcodes `timeout=30.0` is now testing against a stale value. The test still passes — it passes the old default explicitly — so nobody discovers that the new default is not exercised in tests. The test provides false confidence.

2. **Redundant maintenance burden.** A `Typed` class with 8 defaulted fields produces 8 magic numbers that must be synchronized between the class definition and every test that calls the method. When a field is renamed (e.g., `timeout` → `worker_timeout`), every test must be updated manually. With `param_default_values`, the rename propagates automatically.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amazon-science/concurry](https://github.com/amazon-science/concurry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
