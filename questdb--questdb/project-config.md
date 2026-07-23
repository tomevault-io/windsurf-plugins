---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## Project Overview

QuestDB is an open-source time-series database written primarily in zero-GC Java
with native C/C++ libraries for performance-critical operations. It features
column-oriented storage, SIMD-accelerated vector execution, and specialized
time-series SQL extensions.

## Coding guidelines

Java class members are grouped by kind (static vs. instance) and visibility, and
sorted alphabetically. When adding new methods or fields, insert them in the
correct alphabetical position among existing members of the same kind.

Never insert `// ===` or `// ---` banner comments as section headings in any
Java file — not in production code, not in test code. Methods are sorted
alphabetically and will not stay grouped by category.

Use the modern Java 17 features:

- enhanced switch
- multiline string literal
- pattern variables in instanceof checks

However, the java-questdb-client module targets Java 11. When writing code in
the client module, use only legacy Java features.

Whenever dealing with column data, results of expressions, SQL statements, etc.,
always consider what the behavior should be when something is NULL. Be careful
to distinguish NULL as a sentinel value for "not initialized yet" vs. an actual
NULL value.

When choosing a name for a boolean variable, field or method, always use the
is... or has... prefix, as appropriate.

**Log messages must use strictly ASCII characters.** QuestDB's log
infrastructure does not reliably render non-ASCII (e.g., em dashes, curly
quotes, Unicode symbols). Use only plain ASCII punctuation in all `LOG.info()`,
`LOG.error()`, etc. calls.

Use `ObjList<T>` instead of `T[]` object arrays. `ObjList` is QuestDB's
standard resizable list and integrates with `Misc.freeObjList()` /
`Misc.freeObjListIfCloseable()` for resource cleanup.

### Tests

- write all tests using assertMemoryLeak(). This isn't needed for narrow unit
  tests that doesn't allocate native memory.
- resource leaks are a pain point in QuestDB. Always think carefully about all
  possible code paths, especially error paths, and write tests that ensure
  correct resource cleanup on each path.
- assert query results with the fluent `assertQuery(query)` builder
  (`AbstractCairoTest.assertQuery(...)`): `assertQuery(sql).returns(expected)`.
  Chain factory-property assertions as the query warrants — `.timestamp(...)`,
  `.expectSize()`, `.noRandomAccess()`, `.sizeMayVary()`, `.ddl(...)`,
  `.mutateWith(...)`, `.withEngine(...)`, `.withContext(...)`. For execution
  plans use `.assertsPlan(...)` / `.assertsPlanContaining(...)` or fold the plan
  into a data assertion with `.withPlan(...)` / `.withPlanContaining(...)`.
- the old `assertSql(...)` / `TestUtils.assertSql(...)` query-result helpers have
  been REMOVED — `.returns(...)` runs a strictly stronger battery (a second
  cursor pass, a `calculateSize()` cross-check, a variable-column check, and the
  factory-property assertions) that catches bugs the old single-pass print/compare
  silently missed. (`TestServerMain.assertSql(sql, expected)` is a separate
  live-`ServerMain` convenience wrapper and is unrelated.)
- **never use `.returnsOnce(...)` unless the query's projection is genuinely
  non-deterministic across a re-read** — an unseeded `rnd_*` function, or
  time-varying output such as `now()`/`sysdate()`/`systimestamp()`. `returnsOnce`
  deliberately skips the second cursor pass and every check listed above, so for
  any deterministic query it leaves real bugs untested. Default to `.returns(...)`;
  reach for `.returnsOnce(...)` only with a stated reason that the output cannot be
  stable across two reads.
- use execute() to run non-queries (DDL)
- prefer UPPERCASE for SQL keywords (CREATE TABLE, INSERT, SELECT ... AS ... FROM,
  etc.), but mixing cases is acceptable since SQL is case-insensitive
- use a single INSERT statement to insert multiple rows
- use multiline strings for longer statements (multiple INSERT rows, complex
  queries), as well as to assert multiline query results
- use underscore to separate thousands in numbers with 5 digits or more

### QuestDB's SQL dialect

- QuestDB supports multidimensional arrays (e.g., `DOUBLE[]`, `DOUBLE[][]`).
  Dimensionality is encoded in the column type itself, so `DOUBLE[]` and
  `DOUBLE[][]` are distinct column types.
- QuestDB supports the expr::TYPE syntax for casts. Always prefer it to
  CAST(expr, type)
- QuestDB supports underscores as thousands separator: 1_000_000. Always use
  them in numbers of 5 digits or more, and always have that in mind when writing
  implementation code. `Numbers.parseInt()` / `parseLong()` already support
  underscore separators.
- QuestDB does not support DELETE. Rows can only be soft-deleted through
  application logic rules, such as a "deleted BOOLEAN" column.
- QuestDB does support ALTER TABLE DROP PARTITION to mass-delete data.

### Error Position Convention

`SqlException.$(position, msg)` — the position should point at the specific
offending character, not the start of the expression.

## Git & PR Conventions

- **PRs are squash-merged. Commit history on a PR branch is throwaway** — only

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [questdb/questdb](https://github.com/questdb/questdb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
