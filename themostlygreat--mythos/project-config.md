---
trigger: always_on
description: How to write good tests. Use when writing tests in any context — 'write tests', 'add tests', 'test this', 'need tests for', 'improve coverage'. Also consult when writing tests during BDD or debugging. Core knowledge for test quality across all workflows.
---


# Writing Good Tests

Tests prove the system behaves correctly. Every test — unit, integration, E2E, eval — must verify **observable behavior**, not implementation details.

**Core Principle:** TEST BEHAVIOR, NOT IMPLEMENTATION

---

## Philosophy: Behavior-Biased Testing

**What this means:** At every test level, assert on what the system _does_ (outputs, side effects, user-visible outcomes) — never on _how_ it does it (internal state, mock call counts, private methods).

**Why:** Tests coupled to implementation break on every refactor. Behavioral tests survive refactoring because behavior doesn't change — only the internals do.

**Scope preference:** When multiple test types can verify a behavior, prefer the highest scope that's practical. Higher scope = more confidence that the real system works.

```text
Prefer (highest confidence):
  E2E        → proves the user can do the thing
  Integration → proves components work together
  Unit        → proves the algorithm is correct
Fallback (lowest scope):
```

**When to drop to a lower scope:**

- Pure function with many edge cases (20+ combinations) → unit test
- Internal service boundary, no UI involved → integration test
- Algorithm with complex logic (parsing, math, state machines) → unit test
- Only one module's contract matters → integration test

**When to stay at higher scope:**

- User-facing feature or workflow → E2E
- Multiple modules must cooperate → integration or E2E
- "If this breaks, users notice immediately" → E2E

**Announce your decision:** "Test type: [unit/integration/E2E/eval] because [reason]."

For the full decision tree, bug detection matrix, and edge cases: `.safeword/guides/testing-guide.md`

---

## Iron Laws

Non-negotiable at every test level. Violating these produces tests that pass but catch nothing.

**1. Test Behavior, Not Implementation**

```typescript
// WRONG — tests internal state
expect(component.state.count).toBe(1);
expect(mockFn).toHaveBeenCalledWith('internal-detail');

// RIGHT — tests observable behavior
expect(screen.getByText('Count: 1')).toBeVisible();
expect(result).toEqual({ total: 42 });
```

This applies at EVERY level:

- **Unit:** assert on return values, not on which helpers were called
- **Integration:** assert on API responses, not on which service methods fired
- **E2E:** assert on what the user sees, not on DOM structure
- **Eval:** grade the output quality, not the path the LLM took

**2. Every Test Needs a Meaningful Assertion**

If your assertion would pass for ANY input, it asserts nothing.

```typescript
// WRONG — asserts nothing useful
expect(() => processData(input)).not.toThrow();
expect(result).toBeTruthy();
expect(result).toBeDefined();

// RIGHT — asserts specific behavior
expect(processData(input)).toEqual({ status: 'ok', count: 3 });
expect(result.errors).toHaveLength(0);
```

**3. Tests Must Fail First**

A new test that passes immediately is testing nothing — or testing something that already works (no value added). For new behavior: RED then GREEN. For existing code: if a characterization test fails, you found a bug.

**4. One Test, One Behavior**

If a test name has "and" in it, split it. Each test verifies ONE observable outcome.

```typescript
// WRONG
it('validates input and saves to database', ...);

// RIGHT
it('rejects input missing required field', ...);
it('saves valid input to database', ...);
```

**5. Tests Must Be Independent**

No test depends on another test's side effects. Fresh state per test. Run in any order.

---

## Anti-Patterns

The most common ways AI-generated tests go wrong. Watch for all of them.

| Pattern | Problem | Fix |
|---|---|---|
| **Coverage theater** | High line coverage, tests catch no bugs | Every test should fail if you break the behavior it guards |
| **Mock everything** | Tests only verify mock wiring, not real behavior | Use real dependencies where practical; mock only external services |
| **Duplicate tests** | 20 tests with same structure, different values | Use parameterized/table-driven tests: `it.each(...)` |
| **Happy-path only** | Misses edge cases where real bugs live | Always include: empty input, boundary values, error paths |
| **Hardcoded magic values** | Timestamps, IDs, paths break across environments | Use builders, relative values, or factories |
| **Snapshot overuse** | Large snapshots pass review without scrutiny | Prefer targeted assertions; snapshots only for large stable structures |
| **Testing private methods** | Couples tests to implementation | Test through the public API |
| **Exact UI text matching** | Breaks on copy changes | Use regex `/submit/i` or data-testid attributes |
| **Bug-locking** | Tests written against buggy code encode the bug | Write tests BEFORE implementation (TDD), or verify behavior is correct first |
| **Scope defaulting** | AI defaults to unit tests for everything | Ask "what's the highest scope that's practical?" first |

---

## Behavioral Testing by Type

**Unit Tests — Behavioral**

Test the contract (inputs → outputs), not the internals.

```typescript
// Behavioral: asserts on output

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheMostlyGreat/mythos](https://github.com/TheMostlyGreat/mythos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
