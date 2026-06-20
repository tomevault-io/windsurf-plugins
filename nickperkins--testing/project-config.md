---
trigger: always_on
description: |
---


# When to use this skill

Use this skill when you need to:

- Write new tests (Unit, Integration/Feature, or E2E) for any feature or bug fix
- Review existing tests for quality, placement, or value
- Decide whether a test belongs at Unit, Integration, or E2E level
- Refactor tests that are fragile, duplicated, or misplaced
- Design a testing strategy for a new project or module

# When NOT to use this skill

- For non-test code changes that don't need test coverage decisions

---

# The Testing Pyramid (Martin Fowler)

```
         /‾‾‾‾\          E2E: very few, real browser/environment
        /  few  \         Test user JOURNEYS that require real UI/runtime
       /‾‾‾‾‾‾‾‾\
      / moderate  \       Integration/Feature: moderate count, real deps
     / integration \      Test API flows, DB interactions, framework features
    /‾‾‾‾‾‾‾‾‾‾‾‾‾‾\
   /     many fast    \   Unit: many, fast, isolated
  / pure logic & rules \  Test functions, services, helpers — NO external deps
 /________________________\
```

**The pyramid is a COST model.** Each layer up costs more to run, more to maintain, and more to debug when broken. Push tests DOWN the pyramid as far as possible.

---

# Decision Flowchart

When writing a new test, ask these questions in order:

1. **Does it test pure logic with no framework/DB/external dependency?** → Unit test
2. **Does it test an API endpoint, DB interaction, or framework feature?** → Integration/Feature test
3. **Is it already covered by a lower-level test?** → Don't write it again at any level
4. **Does it REQUIRE a real UI/runtime (JS must execute, multi-page state, visual a11y)?** → E2E test
5. **Could you test it with a direct API/function call?** → Integration/Feature test, not E2E

**Default to the lowest layer.** The burden of proof is on E2E: you must not be able to achieve the same confidence at a lower level.

---

# Before Writing Any Test — Adapt to the Project

Testing tools and conventions vary by project. Before writing tests:

1. **Identify the test framework and runner** — e.g., Jest/Vitest (JS), pytest (Python), Pest/PHPUnit (PHP), JUnit (Java), Go `testing`, RSpec (Ruby), etc.
2. **Find existing test files** — look in common locations: `tests/`, `test/`, `spec/`, `__tests__/`, `*_test.*`, `*.test.*`, `*_spec.*`
3. **Mirror existing patterns** — naming, structure, setup/teardown, fixture usage, assertion style
4. **Check project config** — `package.json` scripts, `phpunit.xml`, `pytest.ini`, `Makefile`, CI config for how tests run
5. **If the project has no tests yet** — set up a sensible default structure following the pyramid and conventions of the language/framework

---

# Layer 1 — Unit Tests

**Purpose:** Test pure business logic in isolation. Fast. No framework boot. No database. No external services. No file system.

**MUST test:**

- Pure functions (calculations, transformations, parsing, formatting)
- Business rules and decision logic (eligibility checks, status transitions, validations)
- Data structures (value objects, enums, custom types)
- Utility/helper functions
- Model/domain logic that doesn't require persistence (computed fields, derived values)

**MUST NOT test:**

- Database CRUD operations (that's Integration level)
- HTTP request/response cycles (that's Integration level)
- Framework internals (the framework already tests itself)
- Configuration values (config files ARE the assertion)
- Language/runtime features (e.g., "class exists", "type casting works")

**How to verify placement:** If your unit test needs a database connection, makes HTTP calls, reads files, or boots the full framework, it is NOT a unit test. Move it to Integration.

**Anti-patterns to REJECT:**

```
BAD: Testing framework internals
- Asserting a model's "fillable fields" or "column types"
- Asserting that type casting works
- Asserting that a class exists
- Asserting that a method is defined

BAD: Testing through external dependencies
- Creating DB records to test a pure calculation
- Calling a real API to test response parsing
```

**Correct unit test patterns:**

```
GOOD: Test the business logic directly
- Given inputs X and Y, function returns Z
- Given edge case input, function handles it correctly
- Given invalid input, function throws expected error

GOOD: Test query/command building without executing
- Verify the query builder produces correct SQL
- Verify the command object has correct parameters

GOOD: Test state transitions
- Given status A + event B, new status is C
- Given precondition not met, transition is rejected
```

---

# Layer 2 — Integration/Feature Tests

**Purpose:** Test how components work together. HTTP request/response cycles, database interactions, middleware, authentication/authorization, service integration, API contracts.

**MUST test:**

- API endpoints / controller actions (correct status codes, response shapes, side effects)
- Authorization (who can access what, who is forbidden, who is redirected)
- Input validation (bad input is rejected, good input is accepted)
- Database interactions (creates, updates, deletes produce correct DB state)
- Service integration (calling a service produces correct side effects — emails sent, events dispatched, caches invalidated)
- Multi-tenancy or scoping (tenant A cannot see tenant B's data)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nickperkins/testing](https://github.com/nickperkins/testing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-20 -->
