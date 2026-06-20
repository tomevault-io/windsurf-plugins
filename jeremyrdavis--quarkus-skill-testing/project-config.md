---
trigger: always_on
description: >
---


# Quarkus Testing Skill

Conventions for tests that involve Quarkus — `@QuarkusTest`, the integration-test companion pattern, REST
Assured idioms, and CDI bean mocking. The goal: tests that are fast, deterministic, and unambiguous about what
they're actually exercising.

**Foundational principle.** If you mock the system under test, you're not testing — you're scaffolding. A test class named `XTest` that mocks `X` asserts only what Mockito returned; the rules inside `X` go untested. Mock *collaborators* (other application services, external clients) — never the class the test is named after, never aggregates, never value objects. "We'll test the rules in pure JUnit later" + a `pure-JUnit` file that doesn't exist = the rules go untested forever.

**Red Flags — STOP if you find yourself thinking:**

- About to mock the class your `*Test` class is named after.
- About to mock a domain type (aggregate, value object, entity).
- About to use both `@InjectMock` and `QuarkusMock.installMockForType` for the same bean.
- "We'll write the actual rule tests in pure JUnit later" — and that pure-JUnit file does not exist yet.
- "The `@QuarkusTest` just verifies wiring" — when the test class is named after the rules class.
- Endpoint path in the test doesn't match the resource's `@Path` declaration.

If any of these surface, re-read Core Rules and Excuse / Reality before typing.

---

## When to Use

- Writing or editing a test class annotated with `@QuarkusTest` or `@QuarkusIntegrationTest`.
- Naming a new IT class — use the `*IT extends *Test` companion pattern.
- Deciding whether to use `@InjectMock`, `QuarkusMock.installMockForType`, or `QuarkusMock.installMockForInstance`.
- Mocking a CDI bean (application service, repository).
- Hitting an endpoint with REST Assured.
- Reviewing a test that mocks a `Service` and names the variable `mockRepository`, or that uses both
  `@InjectMock` and `QuarkusMock.installMockForType` for the same bean.

**Out of scope**: pure-JVM aggregate / value-object tests (no Quarkus, just JUnit + assertions — those are
plain unit tests), end-to-end tests against a deployed environment, performance tests.

---

## Core Rules

1. **Use `@QuarkusTest` for JVM tests, `@QuarkusIntegrationTest` for packaged-mode tests.** Quarkus boots once
   per test profile; tests share the same CDI container.
2. **Use the companion pattern for integration tests:** `class FooIT extends FooTest {}`. The IT class adds
   `@QuarkusIntegrationTest` and inherits every test method, so the same suite runs against both the JVM build
   and the packaged artifact. Match this everywhere.
3. **`@InjectMock` is always the default.** Declare `@InjectMock OrderService orders;` and stub in `@BeforeEach`
   with `Mockito.when(...)`. The mock is class-scoped and visible to every test method. Always start here.
   Escalate to `QuarkusMock.installMockForType` only for the specific cases in Rule 4.
4. **Use `QuarkusMock.installMockForType` only for these three cases.** It replaces the bean globally for the
   test's duration; reach for it when `@InjectMock` can't do the job.
   - **(a) Programmatic control.** You need to construct the mock or fake yourself in a setup method — typically
     a hand-rolled stateful fake (a real class, not a Mockito-generated mock) whose construction or wiring is
     too complex for field injection. Install in `@BeforeAll` for the class lifetime, or `@BeforeEach` per test.
     For a non-static `@BeforeAll` method, the test class needs `@TestInstance(Lifecycle.PER_CLASS)`.
   - **(b) Normal-scoped bean replacement.** The bean is `@ApplicationScoped` or `@RequestScoped` and the mock
     must be visible to *other* beans that inject the same type — not just to the field in your test class.
     `installMockForType` swaps the delegate globally so every injection point sees the mock.
   - **(c) Dynamic mocking within a test.** You need to change the mock implementation mid-execution.
     Pair with `QuarkusMock.installMockForInstance` (Rule 5).
5. **`QuarkusMock.installMockForInstance` is only used in conjunction with `installMockForType`.** When an
   individual test method needs to override the already-installed type mock, build a per-test mock and call
   `QuarkusMock.installMockForInstance(perTestMock, installedDefault)`. The override applies for the duration
   of that test method only. Don't use `installMockForInstance` standalone — it overrides nothing if no type
   mock is registered.
6. **`QuarkusMock` is incompatible with parallel test execution.** Both `installMockForType` and
   `installMockForInstance` replace beans globally for the test's duration, which causes race conditions when
   tests run in parallel. If the project enables JUnit parallelism (`junit.jupiter.execution.parallel.enabled`),
   stick to `@InjectMock` for these tests.
7. **Never declare the same bean with both `@InjectMock` and `QuarkusMock.installMockForType`.** That's the
   cargo-cult pattern: two mechanisms install the same mock, one always wins, the other is dead code that
   misleads readers. Pick the right mechanism per Rules 3-4 and use only that one. (This rule does *not*
   forbid the legitimate `installMockForType` + `installMockForInstance` pairing from Rule 5, where the two
   work together by design.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jeremyrdavis/quarkus-skill-testing](https://github.com/jeremyrdavis/quarkus-skill-testing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
