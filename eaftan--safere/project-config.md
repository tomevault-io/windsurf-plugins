---
trigger: always_on
description: SafeRE is a linear-time regular expression matching library for Java, modeled on
---

# SafeRE — Agent Guidelines

## Project Overview

SafeRE is a linear-time regular expression matching library for Java, modeled on
[RE2](https://github.com/google/re2).
The RE2/J (Java) reference is in `re2j-reference/`.

- **Package**: `org.safere`
- **Java version**: 21 (LTS) — built and tested with OpenJDK 25
- **Build**: Maven (`mvn`)
- **Tests**: JUnit 6 (6.0.3), AssertJ
- **Coverage**: JaCoCo
- **Benchmarks**: JMH (Java Microbenchmark Harness)

## License

BSD 3-Clause License (same as RE2 and RE2/J). All source files must
include a license header. Most files use this header:

```java
// This file is part of a Java port of RE2 (https://github.com/google/re2).
// Original RE2 code is Copyright (c) 2009 The RE2 Authors.
// Modifications and Java port Copyright (c) 2026 Eddie Aftandilian.
// Licensed under the BSD 3-Clause License (see LICENSE file).
```

Files that incorporate code from RE2/J use this header instead:

```java
// This file is part of a Java port of RE2 (https://github.com/google/re2).
// Original RE2 code is Copyright (c) 2009 The RE2 Authors.
// Portions derived from RE2/J (https://github.com/google/re2j),
// Copyright (c) 2009 The Go Authors.
// Modifications and Java port Copyright (c) 2026 Eddie Aftandilian.
// Licensed under the BSD 3-Clause License (see LICENSE file).
```

## Build & Test

```bash
# Run tests (quiet output)
mvn -pl safere test -q

# Install to local repo (needed before benchmarks)
mvn install -DskipTests -q

# Run benchmarks (see Benchmarking section below)
./run-java-benchmarks.sh '^org\.safere\.benchmark\.RegexBenchmark\.'
```

## Code Style

Follow the [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html):

- 2-space indentation, no tabs
- 100-character line limit
- Braces on same line (`if (...) {`)
- One class per file (except private inner classes)
- `static` imports grouped separately, sorted alphabetically
- Non-static imports sorted alphabetically
- No wildcard imports
- Use `@Override` on all overriding methods
- Write Javadoc for all public and protected members
- Use `{@code ...}` in Javadoc for code fragments
- Prefer `Objects.requireNonNull` over explicit comparisons to `null` for
  validating required arguments and state, when appropriate.
- Fields: `camelCase`; constants: `UPPER_SNAKE_CASE`; classes: `PascalCase`

## Project Structure

```
safere/src/main/java/org/safere/          # Library source
safere/src/test/java/org/safere/          # Tests
safere-benchmarks/                         # JMH benchmark suite
```

## Architecture

The processing pipeline mirrors RE2:

```
Pattern string → Parse → Simplify → Compile → Execute
                  ↓         ↓          ↓          ↓
               Regexp     Regexp      Prog     NFA/DFA
               (AST)    (simpler)  (bytecode)   (match)
```

### Key Internal Classes

- `Parser` — stack-based operator-precedence regex parser → `Regexp` AST
- `Simplifier` — AST simplification (character class folding, etc.)
- `Compiler` — Thompson NFA construction → `Prog` / `Inst` bytecode
- `Regexp` — AST node (operator + children)
- `CharClass` / `CharClassBuilder` — sorted Unicode code point ranges
- `Prog` / `Inst` — compiled bytecode program

### Execution Engines (in priority order)

1. **Fast paths** — literal `String.indexOf()`, character-class bitmap scan
2. **OnePass** — deterministic single-pass matcher for unambiguous patterns
3. **DFA** — lazy DFA with cached states (forward, reverse, anchored)
4. **BitState** — NFA with visited-state bitmap for small texts
5. **NFA** — Pike VM for arbitrarily large texts

Engine selection in `Matcher.doFind()`:
1. Literal fast path → `String.indexOf()`
2. Anchored OnePass → direct OnePass if `^` and OnePass-eligible
3. Prefix acceleration → skip to first literal/charclass prefix match
4. OnePass for small text → `searchUnanchored()` if text ≤ 256 chars
5. DFA sandwich → forward DFA, reverse DFA, anchored DFA for match bounds
6. BitState/NFA fallback → full search with capture extraction

### Public API

Drop-in replacements for `java.util.regex`:
- `Pattern` — compiled regex (replaces `java.util.regex.Pattern`)
- `Matcher` — match state (replaces `java.util.regex.Matcher`)
- `PatternSet` — multi-pattern matching (SafeRE-only feature)

## Testing

- Use JUnit 6 (6.0.3) with `org.junit.jupiter.api.*` imports
- Use AssertJ (`org.assertj.core.api.Assertions.*`) for all assertions
- Test class naming: `FooTest.java` for `Foo.java`
- Name regression test classes, nested classes, methods, and display names for
  the behavior under test, not the issue number. Do not use names like
  `Issue123RegressionTest`, `Issue123Regressions`, or
  `original issue #123 test case`.
- Keep issue references in comments, Javadocs, or display-name suffixes only
  when they add traceability. The issue number should never be the primary
  description of the test.
- Use `@Test`, `@ParameterizedTest`, `@DisplayName` as appropriate
- Aim for high coverage; JaCoCo is configured in the build
- Port test cases from RE2's C++ test suite where applicable
- For performance regressions, do not hardcode specific elapsed durations in
  tests. Test the performance behavior directly in a way that is stable across

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eaftan/safere](https://github.com/eaftan/safere) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->
