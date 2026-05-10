---
trigger: always_on
description: Java Unit testing guidelines
---

# Java Unit testing guidelines

Effective Java unit testing involves using JUnit 5 annotations and AssertJ for fluent assertions. Tests should follow the Given-When-Then structure with descriptive names for clarity. Each test must have a single responsibility, be independent, and leverage parameterized tests for data variations. Mocking dependencies with frameworks like Mockito is crucial for isolating the unit under test. While code coverage is a useful guide, the focus should be on meaningful tests for critical logic and edge cases. Test classes and methods should typically be package-private. Strategies for code splitting include small test methods and helper functions. Anti-patterns like testing implementation details, hard-coded values, and ignoring failures should be avoided. Proper state management involves isolated state and immutable objects, and error handling should include testing for expected exceptions and their messages.

## Implementing These Principles

These guidelines are built upon the following core principles:

1.  **Clarity and Readability**: Tests should be easy to understand. This is achieved through descriptive names (or `@DisplayName`), a clear Given-When-Then structure, and focused assertions. Readable tests serve as living documentation for the code under test.
2.  **Isolation and Independence**: Each test must be self-contained, not relying on the state or outcome of other tests. Dependencies should be mocked to ensure the unit under test is validated in isolation. This leads to reliable and stable test suites.
3.  **Comprehensive Validation**: Tests should thoroughly verify the behavior of the unit, including its responses to valid inputs, edge cases, boundary conditions, and error scenarios. This involves not just positive paths but also how the code handles failures and exceptions.
4.  **Modern Tooling and Practices**: Leverage modern testing frameworks (JUnit 5), fluent assertion libraries (AssertJ), and mocking tools (Mockito) to write expressive, maintainable, and powerful tests. Utilize features like parameterized tests to reduce boilerplate and improve coverage of data variations.
5.  **Maintainability and Focus**: Tests should be easy to maintain. This means avoiding tests that are too complex, test implementation details, or have multiple responsibilities. A well-written test makes it clear what is being tested and why, simplifying debugging and refactoring efforts.

## Table of contents

- Rule 1: Use JUnit 5 Annotations
- Rule 2: Use AssertJ for Assertions
- Rule 3: Structure Tests with Given-When-Then
- Rule 4: Use Descriptive Test Names
- Rule 5: Aim for Single Responsibility in Tests
- Rule 6: Ensure Tests are Independent
- Rule 7: Use Parameterized Tests for Data Variations
- Rule 8: Utilize Mocking for Dependencies (Mockito)
- Rule 9: Consider Test Coverage, But Don't Obsess
- Rule 10: Test Scopes
- Rule 11: Code Splitting Strategies
- Rule 12: Anti-patterns and Code Smells
- Rule 13: State Management
- Rule 14: Error Handling
- Rule 15: Leverage JSpecify for Null Safety
- Rule 16: Key Questions to Guide Test Creation (RIGHT-BICEP)
- Rule 17: Characteristics of Good Tests (A-TRIP)
- Rule 18: Verifying CORRECT Boundary Conditions

## Rule 1: Use JUnit 5 Annotations

Title: Prefer JUnit 5 annotations over JUnit 4.
Description: Utilize annotations from the `org.junit.jupiter.api` package (e.g., `@Test`, `@BeforeEach`, `@AfterEach`, `@DisplayName`, `@Nested`, `@Disabled`) instead of their JUnit 4 counterparts (`@org.junit.Test`, `@Before`, `@After`, `@Ignore`). This ensures consistency and allows leveraging the full capabilities of JUnit 5.

**Good example:**

```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import static org.assertj.core.api.Assertions.assertThat;

@DisplayName("My Service Test")
class MyServiceTest {

    private MyService service;

    @BeforeEach
    void setUp() {
        service = new MyService(); // Setup executed before each test
    }

    @Test
    @DisplayName("should process data correctly")
    void processData() {
        // Given
        String input = "test";

        // When
        String result = service.process(input);

        // Then
        assertThat(result).isEqualTo("PROCESSED:test");
    }
}
```

**Bad Example:**

```java
import org.junit.Before; // JUnit 4
import org.junit.Test;   // JUnit 4
import static org.junit.Assert.assertEquals; // JUnit 4 Assert

public class MyServiceTest {

    private MyService service;

    @Before // JUnit 4
    public void setup() {
        service = new MyService();
    }

    @Test // JUnit 4
    public void processData() {
        String input = "test";
        String result = service.process(input);
        assertEquals("PROCESSED:test", result); // JUnit 4 Assert
    }
}
```

## Rule 2: Use AssertJ for Assertions

Title: Prefer AssertJ for assertions.
Description: Employ AssertJ's fluent API (`org.assertj.core.api.Assertions.assertThat`) for more readable, expressive, and maintainable assertions compared to JUnit Jupiter's `Assertions` class or Hamcrest matchers.

**Good Example:**

```java
import org.junit.jupiter.api.Test;
import static org.assertj.core.api.Assertions.assertThat;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alkoleft/platform-context-exporter](https://github.com/alkoleft/platform-context-exporter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
