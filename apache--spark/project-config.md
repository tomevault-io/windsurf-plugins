---
trigger: always_on
description: Before the first code edit or running test in a session, ensure a clean working environment. DO NOT skip these checks:
---

# Apache Spark

## Pre-flight Checks

Before the first code edit or running test in a session, ensure a clean working environment. DO NOT skip these checks:

1. Run `git remote -v` to identify the personal fork and upstream (`apache/spark`). If unclear, ask the user to configure their remotes following the standard convention (`origin` for the fork, `upstream` for `apache/spark`).
2. If the latest commit on `<upstream>/master` is more than a day old (check with `git log -1 --format="%ci" <upstream>/master`), run `git fetch <upstream> master`.
3. If there are uncommitted changes (check with `git status`), ask the user to stash them before proceeding.
4. Switch to the appropriate branch:
   - **Existing PR**: resolve the PR branch name via `gh api repos/apache/spark/pulls/<number> --jq '.head.ref'`, then look for a local branch matching that name. If found, switch to it and inform the user. If not found, ask whether to fetch it or if there is a local branch under a different name.
   - **New edits**: ask the user to choose: create a new git worktree from `<upstream>/master` and work from there (recommended), or create and switch to a new branch from `<upstream>/master`.
   - **Running tests**: use `<upstream>/master`.

## Development Notes

SQL golden file tests are managed by `SQLQueryTestSuite` and its variants. Read the class documentation before running or updating these tests. DO NOT edit the generated golden files (`.sql.out`) directly. Always regenerate them when needed, and carefully review the diff to make sure it's expected.

Spark Connect protocol is defined in proto files under `sql/connect/common/src/main/protobuf/`. Read the README there before modifying proto definitions.

Avoid introducing non-ASCII characters in code or comments. String literals may contain non-ASCII when the content requires it (error messages, test data, etc.). Identifiers are ASCII by convention. The common failure mode is typographic characters (em-dash, smart quotes, ellipsis, non-breaking space) sneaking into comments; scalastyle flags some of these. Spot-check before committing: `grep -rn -P "[^\x00-\x7F]" <files>`.

Keep source lines within 100 characters — the linters enforce this for Scala, Java, and Python, and LLMs commonly overrun it in comments and long expressions. A quick scan of just the changed files catches most cases in seconds, far cheaper than a CI round trip:

    { git diff --name-only --diff-filter=ACM HEAD; git ls-files --others --exclude-standard; } \
      | grep -E '\.(scala|java|py)$' | sort -u \
      | xargs -r awk 'length>100 && $0 !~ /^[[:space:]]*(import|package) / && $0 !~ /https?:\/\// \
          {print FILENAME":"FNR": "length" chars"}'

This is only a hint: it approximates the linters' exemptions (imports, URLs) rather than matching them exactly, so it can over- or under-report. The linters remain the source of truth.

## Scala Test Base Classes

When writing a new Scala test suite, pick the lowest base class that provides what the test actually needs. Spark uses the `AnyFunSuite` ScalaTest style throughout, so the bases below are the chain to choose from. Each adds capability on top of the previous:

    SparkFunSuite                                                           (core)
      <- PlanTest                                                           (sql/catalyst)
        <- QueryTest                                                        (sql/core)

| Test scope | Base | Notes |
|------------|------|-------|
| Plain JVM/Scala — no Spark SQL | `SparkFunSuite` | `core` utilities, RDD, network, util classes, etc. Adds per-test timeout, `testRetry`, `gridTest`, thread audit, fixed timezone/locale, `withTempDir`, `withLogAppender`, `checkError`. |
| Catalyst plan tests — no `SparkSession` | `PlanTest` | Adds `comparePlans`, `normalizePlan`, `normalizeExprIds`. For analyzer / optimizer / planner rule tests. |
| SQL/DataFrame tests — needs a `SparkSession` | `QueryTest` | Adds `checkAnswer`, codegen-on/off helpers. `spark: SparkSession` is abstract and must be supplied by a session-providing trait (see below). |

### Providing a `SparkSession` for `QueryTest`

`QueryTest` declares `spark: SparkSession` abstractly via `SparkSessionProvider`, so it cannot be instantiated on its own. A concrete suite mixes in one of the session-providing traits below:

    QueryTest                                                               (abstract `spark`)
      + SharedSparkSession (sql/core)        -> classic in-process `TestSparkSession`
      + TestHiveSingleton  (sql/hive)        -> Hive-backed `TestHive` session

| Session provider | Module / location | Typical usage |
|---|---|---|
| `SharedSparkSession` | `sql/core` | Already extends `QueryTest` for historical reasons, but still mix in `QueryTest` explicitly, e.g. `class X extends QueryTest with SharedSparkSession`. Default for tests under `sql/core`. |
| `TestHiveSingleton` | `sql/hive` | Mixed in alongside `QueryTest`, e.g. `class X extends QueryTest with TestHiveSingleton`. Used by tests under `sql/hive`. |

## Python Test Base Classes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/spark](https://github.com/apache/spark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
