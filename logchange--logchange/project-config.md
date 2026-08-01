---
trigger: always_on
description: - https://www.jetbrains.com/junie/
---

# Guidelines and Engineering Best Practice (Junie)

- https://www.jetbrains.com/junie/

## Core Engineering Principles
- **SOLID:**
  - **Single Responsibility:** Each class/module should have one reason to change. Split responsibilities into focused types.
  - **Open/Closed:** Prefer adding new implementations over modifying stable ones. Use interfaces and composition to extend behavior.
  - **Liskov Substitution:** Subtypes must be usable anywhere their base type is expected. Avoid strengthening preconditions or weakening postconditions.
  - **Interface Segregation:** Prefer small, cohesive interfaces tailored to clients.
  - **Dependency Inversion:** High-level modules depend on abstractions. Pass interfaces or functional types rather than concrete classes.
- **KISS:** Keep solutions as simple as possible, minimize cleverness.
- **YAGNI:** Don’t build features until they’re needed.
- **DRY:** Avoid duplication; extract reusable utilities when duplication appears twice.
- **Fail Fast:** Validate early and throw meaningful exceptions on invalid state.

## Architecture and Structure
- **Modules:** Keep logic in the closest relevant module. Shared functionality belongs in logchange-utils. Domain and generation logic live in the logchange-core. Keep CLI/plugins thin and delegating to logchange-commands.
- **Public API:** Minimize surface area. Keep classes/methods package-private unless truly needed externally.
- **Immutability:** Prefer immutable value objects. Use final fields and defensive copies where applicable.
- **Configuration:** Centralize config parsing/validation. Provide sensible defaults. Make behavior explicit and documented.

## Coding Standards
- **Language:** Java (consistent with project target). Use modern language features when they clearly improve readability and safety.
- **Naming:** Use descriptive names. For tests, use GivenWhenThen in method names.
- **Null-safety:** Avoid returning null. Prefer Optional for optional values; validate inputs with clear exceptions.
- **Exceptions:** Throw specific checked/unchecked exceptions as appropriate. Never swallow exceptions; add context and rethrow. Do not use exceptions for normal control flow.
- **Collections:** Preserve insertion order when it matters for changelog determinism. Use unmodifiable views when exposing collections.
- **Concurrency:** Avoid shared mutable state. If needed, document threading assumptions and use proper synchronization.

## Logging
- Use the project logger: dev.logchange.utils.logger.LogchangeLogger with LoggerLevel.
- Levels
  - ERROR: Failures that abort the operation; include actionable context.
  - WARN: Suspicious or deprecated usage that does not abort.
  - INFO: High-level progress (start/finish of commands, key decisions).
  - DEBUG: Detailed diagnostics helpful for troubleshooting; keep noise controlled.
- Do not log secrets, tokens, or personal data. Redact sensitive paths when necessary.
- Prefer structured, consistent messages. Include identifiers (e.g., version, file paths) to ease debugging.
- Do not use System.out/err directly in production code; tests may capture System.out when verifying CLI output.

## Error Handling
- Validate inputs at boundaries; return useful messages for CLI users and plugin users.
- Add context when rethrowing (include file path, template name, or config key).
- For user-facing commands, convert low-level exceptions to clear messages and non-zero exit codes.

## Testing Philosophy (Use TDD)
- Follow the TDD loop:
  1. Red: Write a failing test describing desired behavior.
  2. Green: Implement the simplest code to pass the test.
  3. Refactor: Improve design while tests stay green.
- Testing priorities: correctness, determinism, readability, speed.
- Tests should be deterministic and hermetic. Avoid external network and time-of-day dependencies. Control time via injected clocks when needed.

## Test Structure and Format
- Frameworks: JUnit 5 (Jupiter). Use Mockito (or hand-rolled fakes) for collaborators.
- Naming
  - Class names: <ClassUnderTest>Test or <Feature>IntegrationTest
  - Method names: given_<context>_when_<action>_then_<outcome>
- Given-When-Then layout within tests; keep clear separation with blank lines/comments.
```java
class SomeTest {

    @Test
    void given_OrderWithItem_when_CalculateTotal_then_ReturnsTotal() {
        // given: 
        Order order = new Order();
        order.addItem(new OrderItem("item1", 10));

        // when:
        int total = service.calculateTotal(order);

        // then: 
        assertThat(total).isEqualTo(25);
    }
}
```
- Assertions: Prefer expressive assertions. Assert full outputs (e.g., generated CHANGELOG content) using golden files only when necessary and stable.
- Coverage: Focus on meaningful scenarios and edge cases (empty/unreleased entries, malformed YAML, missing templates, conflicting versions, ordering rules).
- Unit vs Integration
  - Unit tests: exercise one class or small unit with collaborators mocked/faked.
  - Integration tests: verify end-to-end flows (e.g., generating changelog from sample directories and templates). Place inputs in src/test/resources and compare to expected outputs (e.g., expected-CHANGELOG.md).
- Test Data
  - Keep fixtures minimal and documented. Reuse helpers for building test objects.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [logchange/logchange](https://github.com/logchange/logchange) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
