---
trigger: always_on
description: - Frameworks: xUnit, Moq, AwesomeAssertions (FluentAssertions syntax, different namespace).
---

# Unit Testing Rules (xUnit + Moq + AwesomeAssertions)

- Frameworks: xUnit, Moq, AwesomeAssertions (FluentAssertions syntax, different namespace).
- Use Moq for test doubles; do not introduce fake or stub implementations.
- Moq callbacks should only be used when verifying the same behavior is impossible with `Verify`.
- Do not add comments to test code (especially not Arrange/Act/Assert).
- Braces must never be omitted.
- Expression-bodied members are not permitted in tests.

## Naming

- Test class name: `<ClassName>Tests`
- Test namespace mirrors the product namespace with `.Test` inserted before the final segment.  
  Example:  
  - Product: `Lantean.QbtMud.Torrent.Services`  
  - Tests: `Lantean.QbtMud.Torrent.Test.Services`
- Test method names use Given-When-Then:  
  `GIVEN_StateOfItem_WHEN_PerformingOperation_THEN_ShouldBeExpectedState`

## Test Class Structure

- For non-component unit tests, use a readonly field named `_target` only when one shared constructor setup genuinely serves most tests in the class.
- When `_target` is used, construct it in the test class constructor.
- When constructor arguments, dependency behavior, or lifetime vary per test and that local construction is the common case for the class, do not force a class-level `_target`; construct the system under test inside each test and store it in a method-local variable named `target`.
- Local `target` instances are allowed in classes that also use `_target` when they are the minority case and the altered construction is part of the scenario under test.
- Mocks used across tests are private readonly fields created with `Mock.Of<T>()`.
- Mocks that are local to a single test method should use `new Mock<T>()`.

## Test Data Conventions

- Strings use the property name as the value (no `nameof`), e.g. `user.FirstName = "FirstName"`.
- Dates use a fixed point in time: `2000-01-01 00:00` with the correct `DateTimeKind`.  
  Adjust earlier/later than this when ranges or ordering are required.
- Numeric values must be contextually appropriate.

## Coverage and Access

- Tests must cover 100% of the lines and branches of the implementation.
- Never use reflection to invoke implementation code. Cover private or protected methods via normal execution flow only.
- If code cannot be reached via public methods then consider asking to refactor.
- Do not add test-only hooks or methods to production code. If coverage gaps exist, ask for a refactor to expose the behavior through public/UI flows.

## Clarification Policy

- Do not make assumptions. If any referenced code or behavior is unclear, ask for clarification before writing tests.

## Line endings
- Use CRLF line terminators for any files you write or modify.
- After editing any test file that is expected to use CRLF, run `unix2dos <changed files>` to normalize the entire file and eliminate any LF or mixed endings introduced by patching tools.
- Do not run `unix2dos` on files that are intentionally LF per `.gitattributes` (for example `*.sh`, `*.bash`, `*.py`, and `justfile`).
- Before finishing, verify every changed CRLF-governed file is `crlf` and not `mixed`.

## Formatting
- After modifying test files, run `dotnet format --include <changed files> --artifacts-path=/tmp/artifacts/qbtmud` for the files changed in the current task only.
- Do not format unrelated files.

## Test Execution

- After each behavior-affecting set of changes, follow the test execution instructions in `AGENTS.md`.
- If the change is docs-only/report-only/markdown-only and does not affect behavior, test execution is optional unless explicitly requested.
- When contributing PR summaries or PR bodies, describe testing in terms of the coverage added or updated by the change (for example, new service tests, component regressions covered, migrated scenarios validated), not just the commands executed.

## Anti-Smell Rules

- Do not inspect invocation internals in assertions. Avoid `Invocations.Count/Any/Where/Single/First/Last`, `Method.Name`, and `Arguments[...]` in test assertions.
- Prefer `Verify(...)` for Moq assertions and bUnit planned-invocation assertions for JS interop (`Setup/SetupVoid(...).Invocations`) when call counting is required.
- If invocation history must be reset between phases, use the shared `ClearInvocations()` helper in test infrastructure, not ad-hoc invocation-list manipulation.
- Do not assert UI text by scanning component markup strings (`Markup.Should().Contain/NotContain`, `Markup.IndexOf`).
- Prefer component- and attribute-based assertions (`FindComponent`, `FindComponentByTestId`, strongly-typed component state/properties).
- Do not use `Task.Delay(...)` for synchronization or timing in tests.
- Use deterministic waiting primitives instead (`WaitForAssertion`, `WaitForState`, `WaitAsync(timeout)`, `Task.Yield` polling only when strictly necessary).

## Blazor Components

- Use `bUnit` for testing Blazor components.
- Follow the same naming conventions as above.
- Render the component under test inside each test by default.
- Store the rendered component under test in a method-local variable named `target`.
- Use `TestContext` for component rendering and dependency injection.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lantean-code/qbtmud](https://github.com/lantean-code/qbtmud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
