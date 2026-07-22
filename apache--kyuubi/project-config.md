---
trigger: always_on
description: A guide for AI coding agents contributing to Apache Kyuubi. Pairs with `CONTRIBUTING.md` and the [Contributor Guide](https://kyuubi.readthedocs.io/en/master/contributing/code/index.html).
---

# Apache Kyuubi — Agent Instructions

A guide for AI coding agents contributing to Apache Kyuubi. Pairs with `CONTRIBUTING.md` and the [Contributor Guide](https://kyuubi.readthedocs.io/en/master/contributing/code/index.html).

## Pre-flight Checks

Before the first edit or test in a session:

1. Run `git remote -v`. An `apache` remote must point to `apache/kyuubi`; do not work from a fork-only checkout.
2. If `apache/master` is stale, `git fetch apache master` before branching or rebasing.
3. Check `git status`. If the tree is dirty, ask the user to stash before any branch switch or release/RAT-style check.
4. Existing PR: resolve the branch via `gh api repos/apache/kyuubi/pulls/<num> --jq '.head.ref'`, then check it out.
5. New work: branch from `apache/master`. Branch names like `kyuubi-NNNN-short-slug` are convention, not policy.
6. Confirm `git config user.email` matches an email on the GitHub account that will open the PR.
7. Keep private/ignored files outside the repo root — RAT scans generated local files even when `.gitignore`d.

## Architecture

Kyuubi is a multi-tenant gateway that fronts pluggable SQL engines (Spark, Flink, Trino, JDBC, etc.). The module split is load-bearing in code review.

### Module Layout

- `kyuubi-server/` — gateway process. Must not depend on engine modules under `externals/` or implement engine-side runtime behavior.
- `kyuubi-common/` — shared service abstractions, config registry, session/operation base classes, relocated Hive Thrift RPC.
- `kyuubi-ha/`, `kyuubi-events/`, `kyuubi-metrics/`, `kyuubi-rest-client/`, `kyuubi-zookeeper/` — focused libraries.
- `kyuubi-ctl/` — admin CLI.
- `kyuubi-hive-jdbc/`, `kyuubi-hive-beeline/` — JDBC client and shell.
- `externals/kyuubi-{spark,flink,hive,trino,jdbc,data-agent}-sql-engine/` — engine processes. Must not depend on each other.
- `extensions/{server,spark,flink}/` — opt-in plug-ins.
- `integration-tests/` — cross-module suites.
- `kyuubi-assembly/` — binary distribution.

### Hard Boundaries

- Server must not depend on Kyuubi engine modules or use engine runtime APIs (Spark/Flink internals) to implement engine behavior server-side.
- Protocol/client libraries are not a blanket exception — if one is needed in `kyuubi-server`, keep it limited to wire-protocol or client-model translation and verify the dependency tree.
- Engines must not depend on sibling engines (e.g. no `kyuubi-flink-sql-engine` → `kyuubi-spark-sql-engine`).
- Public APIs are abstract over the cluster manager. Use `killApplication`, not `closeYarnJob`; APIs, classes, and config keys must work for YARN, Kubernetes, and future managers.
- Spark SQL syntax extensions need maintainer agreement, version-aware parser support, and a clear reason they cannot stay upstream-compatible.
- Do not break the Thrift wire protocol. Kyuubi speaks Hive TCLIService through relocated dependencies; never remove or renumber wire fields.
- Prefer reusing existing Thrift-defined operations with an operation-level config key over extending the wire schema; change the wire schema only with maintainer agreement.

### High-Sensitivity Areas

Get reviewer attention before changing:

- `kyuubi-common/.../session/SessionManager`, `.../operation/OperationManager` — lifecycle, concurrency, shutdown.
- `kyuubi-server/.../engine/KubernetesApplicationOperation` — cluster-manager integration; tests must not assume a real cluster.
- `kyuubi-common/.../config/KyuubiConf` — config registry; changes require regenerating `settings.md`.
- `kyuubi-server/.../api/v1/` — public REST surface; add auth checks before exposing.

## Build and Test

Use the bundled Maven wrapper (`build/mvn`).

```
build/mvn -Pfast clean package -DskipTests           # local compile, skips tests/style/docs/RAT/downloads
build/mvn clean package -pl kyuubi-common -am -DskipTests
build/mvn clean install                              # all tests
build/mvn clean install -pl kyuubi-common -am        # one module's tests
build/mvn test -pl kyuubi-server -am -Dtest=none \
    -DwildcardSuites=org.apache.kyuubi.server.api.v1.SessionsResourceSuite
build/mvn test -pl kyuubi-hive-jdbc -am -Dtest=KyuubiStatementTest -DwildcardSuites=none
```

Use `-am` (also-make) when building or testing a single module — without it, Maven fails unless the dependency's artifact is already in `~/.m2`. Integration-style tests that exercise a packaged engine require `build/dist` first.

### Engine profile matrix

| Profile | Notes |
|---|---|
| `-Pspark-3.5` (default), `-Pspark-{3.3,3.4,4.0,4.1,master}` | Spark version |
| `-Pflink-1.20` (default), `-Pflink-{1.17,1.18,1.19}` | Flink version |
| `-Pscala-2.13` | Scala 2.13 (default is 2.12) |
| `-P{spark,flink,hive}-provided` | skip bundled engine downloads |
| `-Pmirror-cdn` | use Apache mirror CDN for engine archives |
| `-Pfast` | skip tests/style/docs/enforcer/RAT/downloads |

When engine code varies across versions, gate source/binary differences by Maven profile and runtime capability differences by feature detection — not by parsing version strings.

### Style, dependencies, docs

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/kyuubi](https://github.com/apache/kyuubi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
