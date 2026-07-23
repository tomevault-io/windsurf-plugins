---
trigger: always_on
description: provides context.
---

# CLAUDE.md

Guidance for Claude Code when working in the Velox repository.

## Branch Hygiene

Before creating a new feature branch, always:

1. `git checkout main`
2. `git fetch upstream && git rebase upstream/main`
3. Delete stale feature branches.
4. Push main to origin if behind upstream.
5. `git checkout -b <new-branch>`

## PR Review

When asked to review a PR (via `/pr-review`), always use the /pr-review skill.

### Review scripts

Use `scripts/review/fetch.py` and `scripts/review/post.py` for PR reviews
instead of raw `gh api` calls.

```bash
# Fetch PR metadata, diff, comments, and reviews in one shot.
python3 scripts/review/fetch.py <owner/repo> <pr-number>
python3 scripts/review/fetch.py <github-pr-url>

# Post a review from a file.
python3 scripts/review/post.py <owner/repo> <pr-number> <event> <body-file>
python3 scripts/review/post.py <github-pr-url> <event> <body-file>
# Events: APPROVE, REQUEST_CHANGES, COMMENT
```

Always draft the review body in `/tmp/` and get approval before calling
`post.py`.

### Review style

See [scripts/review/REVIEW_GUIDE.md](../scripts/review/REVIEW_GUIDE.md).

## Queries

When asked a question about the PR or codebase (via `/query`), use the /query skill.

## Overview

Velox is an open source C++ library for composable data processing and
query execution. Licensed under Apache 2.0. Requires C++20, GCC 11+ or
Clang 15+.

## Build

```bash
make debug    # debug build
make release  # optimized build
```

## Testing

```bash
make unittest                    # run all tests
cd _build/debug && ctest -j 8   # run all tests in parallel
ctest -R ExprTest                # run tests matching a pattern
```

Test files live in `tests/` subdirectories alongside source.

### Grouped tests

Four test suites use `velox_add_grouped_tests` to reduce link times on Linux CI
by batching source files into shared binaries:
- `velox/exec/tests` (`velox_exec_test`, `velox_exec_util_test`)
- `velox/functions/prestosql/aggregates/tests`
- `velox/common/caching/tests`
- `velox/serializers/tests`

All other test suites use individual binaries on all platforms.

On macOS, grouping is off by default (`VELOX_ENABLE_GROUPED_TESTS=OFF`) and each
test file gets its own binary (e.g., `ValuesTest.cpp` → `velox_exec_test_ValuesTest`).
On Linux CI, grouping is on (`velox_exec_test_group0` through `_group7`).
Override with `-DVELOX_ENABLE_GROUPED_TESTS=ON/OFF`.

### Common test workflows

```bash
# Run all test binaries whose ctest name matches a regex.
# On Linux this matches velox_exec_test_group0 … _group7.
# On macOS this matches velox_exec_test_ValuesTest,
# velox_exec_test_HashJoinTest, etc.
cd _build/debug && ctest -R velox_exec

# Run a specific test file (macOS — individual binary)
_build/debug/velox/exec/tests/velox_exec_test_ValuesTest --gtest_filter="ValuesTest.*"

# Run a specific test case (Linux — grouped binary)
_build/debug/velox/exec/tests/velox_exec_test_group3 --gtest_filter="ValuesTest.empty"
```

**Re-running a CI failure locally:** CI reports a failure in
`velox_exec_test_group3` with `ValuesTest.empty`. On Linux, run the grouped
binary directly. On macOS, the grouped binary does not exist — use the
per-file binary instead: `velox_exec_test_ValuesTest --gtest_filter="ValuesTest.empty"`.

**Adding a new test to a grouped suite:** Add the source file to the `SOURCES`
list in the relevant `velox_add_grouped_tests()` call in `CMakeLists.txt`. It
is automatically assigned to a group on Linux and gets its own binary on macOS.

**Creating a new test suite:** Use `velox_add_grouped_tests` for suites with
many test files (10+) that link against large libraries like velox core — each
individual binary pays the full link cost, so grouping them into shared binaries
significantly reduces total CI build time. For suites with only a few test files
or lightweight dependencies, use `add_executable` / `add_test`.

## Formatting

```bash
make format  # format all changed files
```

## Coding Style

Read [CODING_STYLE.md](../CODING_STYLE.md) for the complete guide. Key rules
are summarized below.

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

### Naming Conventions

- **PascalCase** for types and file names.
- **camelCase** for functions, member and local variables.
- **camelCase_** for private and protected member variables.
- **snake_case** for namespace names and build targets.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [facebookincubator/velox](https://github.com/facebookincubator/velox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
