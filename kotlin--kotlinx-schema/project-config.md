---
trigger: always_on
description: This document is for autonomous agents and AI copilots contributing code to this repository. Follow these rules to keep
---

# AGENTS: Development Guidelines for AI Contributors

This document is for autonomous agents and AI copilots contributing code to this repository. Follow these rules to keep
changes safe, comprehensible, and easy to maintain.

## Prime directives

1. Tests first, always.
    - Before changing code, identify or add tests that express the desired behavior.
    - Prefer readable, minimal tests over clever ones. Tests are documentation.
2. Keep tests simple and explicit.
    - Arrange/Act/Assert structure; avoid hidden magic and overuse of helpers.
    - Prefer concrete inputs/outputs; avoid randomness and time dependence.
3. Uphold SOLID principles in production code:
    - Single Responsibility: each class/function should do one thing well.
    - Open/Closed: extend via new code, avoid risky edits to stable code paths.
    - Liskov Substitution: honor contracts; keep types substitutable.
    - Interface Segregation: keep abstractions small and focused.
    - Dependency Inversion: should depend on abstractions, not concretions.
4. Make the minimal change that satisfies the tests and the issue.
5. Keep the build green. Do not merge changes that break existing tests.
6. Prefer clarity over micro-optimizations and cleverness.
7. Ask when uncertain. If requirements are ambiguous, request clarification with a concise question.
8. Write code with the quality of a Kotlin Champion.
9. Prefer using MCP servers like `jetbrains` and `intellij-index` to work with code
10. Don't use MCP to run terminal commands.
11. Suggest updating AGENTS.md/CLAUDE.md with best practices and guidelines.

## Code Style

### Kotlin

- Follow Kotlin coding conventions
- Use the provided `.editorconfig` for consistent formatting
- Use Kotlin typesafe DSL builders where possible and prioritize fluent builders style over standard builder methods.
  If DSL builders produce less readable code, use standard setter methods.
- Prefer DSL builder style (method with lambda blocks) over constructors, if possible.
- Use Kotlin's `val` for immutable properties and `var` for mutable properties. Consider using `lateinit var` instead of
  nullable types, if possible.
- Use multi-dollar interpolation prefix for strings, where applicable
- Use fully qualified imports instead of star imports
- Ensure to preserve backward compatibility when making changes
- Use `//region region_name` / `//endregion` comments to group related members within a class or file
  (e.g. `//region Filtering`, `//region Test cases`). This enables IDE code folding and makes structure scannable
  at a glance. Keep region names short and descriptive.

## Testing guidance

- Write comprehensive tests for new features
- **Prioritize test readability**
- Avoid creating too many test methods. If multiple inputs can be tested against the same logic, use a parameterized
  test instead of repeating `@Test` methods.
- **Prefer parameterized tests** for any scenario with 3+ input/output variations. Choose the source that makes
  the test easiest to read:
  - `@CsvSource` — for short scalar values (String, Boolean, Int, Enum) with no need for grouping. Inline and
    requires no separate provider method.
  - `@MethodSource` — when cases need complex types (collections, data classes), grouping comments, or descriptive
    `override fun toString()` names. Annotate the test class with `@TestInstance(TestInstance.Lifecycle.PER_CLASS)`
    so provider functions are plain instance methods — no `companion object` or `@JvmStatic` needed.
  - Prefer `@MethodSource` over `@CsvSource` when the inline strings would be long or hard to scan.
  - Keep non-parameterizable structural tests (e.g. null handling, type filtering) as plain `@Test`.
- When running tests on a Kotlin Multiplatform project, run only JVM tests,
  if no asked to run tests on another platform too.
- Use function `Names with backticks` for test methods in Kotlin, e.g. "fun `should return 200 OK`()"
- Avoid writing KDocs for tests, keep code self-documenting
- Use Kotest JSON assertions for json and json schema comparisons:
    - Example: `schema shouldEqualJson """ { ... } """.trimIndent()`
    - For Kotlin raw strings containing JSON Schema keywords starting with `$`, escape with Kotlin interpolation escape
      in tests: use $$""" and `${'$'}` inside raw strings where needed, e.g. `${'$'}id`, `${'$'}defs`, `${'$'}ref`.
- Prefer verifying both forms where applicable:
    - `KClass<T>.jsonSchemaString`
    - `KClass<T>.jsonSchema` (JsonObject parsed from the string)
- Keep test JSON readable:
    - Use the `// language=json` comment before multiline JSON blocks for IDE support.
    - Avoid brittle whitespace assertions; compare by structure using `shouldEqualJson`.
- Cover typical scenarios when modifying the generator/introspector:
    - Primitives, enums, nullable properties, lists/maps, nested objects, generics (star-projection to kotlin.Any), and
      description propagation from @Description.
- Ensure non-annotated classes do not gain generated extensions.
- Write Kotlin tests with [kotlin-test](https://github.com/JetBrains/kotlin/tree/master/libraries/kotlin.test),
  [mockk](https://mockk.io/) and [Kotest-assertions](https://kotest.io/docs/assertions/assertions.html)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kotlin/kotlinx-schema](https://github.com/Kotlin/kotlinx-schema) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
