---
trigger: always_on
description: gotest is a code-generation framework for Go test suites.
---

# gotest — Agent Reference

gotest is a code-generation framework for Go test suites.
You write suite structs with lifecycle methods and test cases; the `gotest generate` command produces the test harness (test functions, fixture initialization, lifecycle wiring).
The generated code is not hand-edited.

All assertions accept both `*gotest.T` and `*testing.T` as first argument.

## Rules

These override any default instincts from stdlib `testing` or other Go test frameworks.

**Never call `t.T().Helper()` when using gotest assertions.**
gotest automatically resolves assertion failures to the correct call site in your test code.
Calling `t.T().Helper()` is counterproductive — it removes useful location information from failure output.
Without it, failures show both where the assertion was called (inside the helper) and where the helper was called (in the test).
This applies to all gotest assertion functions (`gotest.Equal`, `gotest.True`, etc.).
If you pass `t.T()` to third-party code that uses Go's standard `t.Errorf`, Go's `t.Helper()` convention applies as normal.

**Never call `t.T().Cleanup()` inside suite methods.**
Use `BeforeAll`/`AfterAll` for suite-level resources and `BeforeEach`/`AfterEach` for per-test resources.
The generated harness manages cleanup ordering (LIFO).
Calling `t.T().Cleanup()` inside suite methods breaks the predictable lifecycle.

**Never call `t.T().Fatalf()` in reusable test helpers.**
Use gotest assertions instead.
Helpers that call `t.T()` panic inside `Eventually`/`Consistently` because the collecting T has no underlying `*testing.T`.
This is the most common source of runtime panics in gotest code.

**Never use `t.T().Skip()` for environment gating.**
Use `SuiteGuard() string` instead.
SuiteGuard runs before shared fixture wiring and `t.Parallel()`, avoiding wasted work.
`t.T().Skip()` inside `BeforeAll` runs after those expensive operations.

**`Nil`/`NotNil` assertions do not exist.**
This is deliberate.
Go generics cannot constrain to "nillable types", so a generic `NotNil(t, x)` would silently accept non-nillable types (e.g. `int`, `struct{}`), creating logical bugs with no compile-time warning.
Use `NotZero`/`Zero` for pointer, interface, and channel nil checks (these types satisfy `comparable`).
For slices, maps, and channels, prefer `Empty`/`NotEmpty` — a nil slice and an empty slice are equivalent for most test intent.
Use `True(t, x != nil)` only when the nil vs empty distinction actually matters.

**`HasPrefix`/`HasSuffix` assertions do not exist.**
Use `Regexp(t, "^prefix", str)` or `Regexp(t, "suffix$", str)`.

## Assertions

All assertion functions call `FailNow()` on failure (fatal — test stops immediately).
All accept optional trailing `msgAndArgs ...any` for custom error messages.

### Unconditional Failure

```go
gotest.Fail(t, msgAndArgs ...any)                    // immediately fails with message
```

Use in unreachable branches (`default` in exhaustive switches, after calls that must not return).
Prefer over `gotest.True(t, false, "msg")`.

### Equality

```go
gotest.Equal[V any](t, expected, actual V)          // deep equality (reflect.DeepEqual)
gotest.NotEqual[V any](t, expected, actual V)        // deep inequality
```

### Boolean

```go
gotest.True(t, value bool)
gotest.False(t, value bool)
```

### Zero Value (also covers nil)

```go
gotest.Zero[V comparable](t, value V)               // value == zero value for type
gotest.NotZero[V comparable](t, value V)             // value != zero value for type
```

`comparable` includes: pointers, interfaces, channels, numerics, strings, bools, structs of comparables, arrays of comparables.
It excludes: slices, maps, functions.

For nil checks on slices, maps, or functions: `gotest.True(t, x != nil)`.

### Collections

```go
gotest.Empty(t, object any)                          // nil, or len == 0 (string, slice, map, chan, array)
gotest.NotEmpty(t, object any)
gotest.Len(t, object any, length int)                // exact length check
gotest.Contains(t, haystack, needle any)             // substring (string), element (slice/array), key (map)
gotest.NotContains(t, haystack, needle any)
gotest.ElementsMatch[V comparable](t, a, b []V)     // same elements, any order
gotest.Subset[V comparable](t, list, subset []V)     // every subset element exists in list
```

### Errors

```go
gotest.NoError(t, err error)                         // err == nil
gotest.Error(t, err error)                           // err != nil
gotest.ErrorIs(t, err, target error)                 // errors.Is(err, target)
gotest.ErrorAs[E error](t, err error) E              // errors.As — returns matched error
gotest.ErrorContains(t, err error, substr string)    // err != nil && strings.Contains(err.Error(), substr)
```

### Comparison

```go
gotest.Greater[V cmp.Ordered](t, a, b V)            // a > b
gotest.GreaterOrEqual[V cmp.Ordered](t, a, b V)     // a >= b
gotest.Less[V cmp.Ordered](t, a, b V)               // a < b
gotest.LessOrEqual[V cmp.Ordered](t, a, b V)        // a <= b
```

### Strings & Patterns

```go
gotest.Regexp[P regexpPattern](t, rx P, str string)  // accepts string pattern or *regexp.Regexp
```

Use `"^prefix"` for HasPrefix, `"suffix$"` for HasSuffix, `"^exact$"` for exact match.

### Numeric

```go

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mvrahden/go-test](https://github.com/mvrahden/go-test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
