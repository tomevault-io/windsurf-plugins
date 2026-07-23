---
trigger: always_on
description: provides context.
---

# CLAUDE.md

Guidance for Claude Code when working in the Axiom repository.

## Overview

Axiom provides reusable SQL query processing components compatible with
[Velox](https://github.com/facebookincubator/velox). Licensed under
Apache 2.0.

## Build

```bash
make debug    # debug build
make release  # optimized build
```

## Testing

```bash
make unittest                    # run all tests
cd _build/debug && ctest -j 8   # run all tests in parallel
ctest -R AggregationParserTest   # run tests matching a pattern
```

Test files live in `tests/` subdirectories alongside source. Place new
tests next to related existing tests, not at the end of the file. Group
tests by topic (e.g., place `tryCast` next to `types`, `notBetween` next
to `ifClause` which uses `between`).

Prefer gtest container matchers over individual assertions:

```cpp
// ❌ Avoid - multiple individual assertions
EXPECT_EQ(result.size(), 3);
EXPECT_EQ(result[0], "a");
EXPECT_EQ(result[1], "b");
EXPECT_EQ(result[2], "c");

// ✅ Prefer - single matcher assertion
EXPECT_THAT(result, testing::ElementsAre("a", "b", "c"));
```

Common matchers:
- `ElementsAre(...)` - exact ordered match
- `UnorderedElementsAre(...)` - exact unordered match
- `Contains(...)` - at least one element matches
- `IsEmpty()` - collection is empty
- `SizeIs(n)` - collection has n elements

Requires `#include <gmock/gmock.h>`.

### Verifying bug fixes

When fixing a bug, write the test **first** and confirm it reproduces the issue before applying the fix. If the test was written after the fix, temporarily revert the fix and verify the test fails without it. A test that passes with and without the fix proves nothing.

### Verifying changes empirically

Do not rely solely on static code analysis (reading code and reasoning about behavior). Always verify changes by running the code:
- Add temporary print statements to confirm the code path is exercised.
- Write a small test to observe actual behavior.
- Remove debug logging before committing.

## Formatting

```bash
make format  # format all changed files
```

## Coding Style

Axiom follows the Velox coding style. See
[CODING_STYLE.md](https://github.com/facebookincubator/velox/blob/main/CODING_STYLE.md)
for the complete guide. Key rules are summarized below.

### Comments

- Use `///` for public API documentation (classes, public methods, public members).
- Use `//` for private/protected members and comments inside code blocks.
- Start comments with active verbs, not "This class…" or "This method…".
  - ❌ `/// This class builds query plans.`
  - ✅ `/// Builds query plans.`
- Comments should be full English sentences starting with a capital letter and ending with a period.
- Comment every class, every non-trivial method, every member variable.
- Do not restate the variable name. Either explain the semantic meaning or omit the comment.
  - ❌ `// A simple counter.` above `size_t count_{0};`
- Avoid redundant comments that repeat what the code already says. Comments should explain *why*, not *what*.
- Use `// TODO: Description.` for future work. Do not include author's username.
- Do not duplicate comments between `.h` and `.cpp`. Document the function in the header; the implementation should not repeat the same comment. Duplicated comments diverge over time.
- Don't leave reasoning-scaffolding comments in committed code. Comments that capture the thought process — naming a sibling function, citing an example from a design discussion ("e.g. for COUNT-style aggregates"), framing the code defensively against an alternative path that was considered and rejected — are useful working notes while writing but rot quickly: they couple to other functions' internals and to ephemeral conversation context that a fresh reader doesn't share. Before committing, delete any comment that names another function and contrasts it ("X does Y; we do Z"), cites a specific operator/aggregate from discussion, describes a path you didn't take, or reads as a running narrative. Keep only comments that describe an invariant the code maintains or a non-obvious constraint a reader must know.
- After trimming, re-read the function as a fresh reader and check the *other* failure mode: stripping every comment can leave branches whose intent is no longer visible. A silent `else` to a non-trivial `if/else-if` chain, a state mutation whose justification depends on a downstream call, a deliberate-looking "do nothing" — these usually need a one-line `// why` even after every scaffolding comment is gone. Brevity is not "zero comments"; it is "no comment that doesn't earn its line."

#### Public class docs — readability

The above rules are mechanical (form). Class-level documentation has an
additional readability requirement, since header docs are read cold by
people who don't know the surrounding design.

**For non-trivial classes** — defined as ANY of:
- Has multi-state behavior (an enum field, mode flag, or `kind` tag the implementation dispatches on)
- Carries invariants beyond defaults (constructor assertions, ordering requirements, lifecycle rules)
- Participates in a larger protocol or algorithm with other classes
- Is a factory / builder / registry / context object callers compose

— the class-level doc MUST include:
- **(a)** ONE sentence stating what the class is (lead, not buried)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [facebookincubator/axiom](https://github.com/facebookincubator/axiom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
