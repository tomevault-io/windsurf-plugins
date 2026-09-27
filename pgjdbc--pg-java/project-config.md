---
trigger: always_on
description: Guidance for AI agents and human contributors working in this repository.
---

# AGENTS.md

Guidance for AI agents and human contributors working in this repository.
Read this first; it is the contract. `CLAUDE.md` just points here.

## Project overview

`pg-java` is a modern, PostgreSQL-specific database driver for the JVM. The
near-term focus is a clean, idiomatic, PostgreSQL-native API. JDBC compliance is
a long-term goal layered on top of the native API. JDBC must not dictate the
shape of the core driver: keep `java.sql.*` out of `postgresql-client` and
`postgresql-client-protocol`.

## Where things are

Read the relevant doc before changing behavior it governs. Do not re-derive a
decision that already has an ADR.

| Path | What it is |
| --- | --- |
| `docs/adr/` | Numbered, canonical architecture decisions (ADR-0001..). `docs/adr/README.md` is the index. |
| `docs/plans/overall.md` | Master implementation plan. Every numbered item (`C0.1`, `N5.7`, `P4`) is one atomic commit; checkboxes track progress. |
| `docs/follow-up.md` | Index of deferred work, with the reason and the files involved. |
| `docs/plans/*.md` | Per-effort plans (performance, bench module, compat suites). |
| `docs/jdbc-conformance-matrix.md`, `docs/pooler-compatibility.md` | Behavior matrices. |
| `docs/api-surface.md` | The enumerated public API surface (ADR-0020), enforced by `ApiSurfaceManifestTest`. A new public type fails the build until it is listed as Stable or Experimental. |
| `docs/static-analysis.md` | Which analysers the build runs and why, which were rejected, and the scope agreed for the ones not yet adopted. |
| `docs/life-of-a-query.md` | Best single orientation doc for the core execution path. |
| `docs/reviews/`, `docs/benchmarks/` | Findings from past audits; benchmark reference numbers. |
| `compat-suites/` | pgjdbc and Hibernate upstream suites run against our driver, with committed baselines. |
| `scripts/` | Integration matrix, benchmark, and Docker helper scripts. |

ADRs that most often bind a change: ADR-0001 (I/O and concurrency),
ADR-0002 (module boundaries), ADR-0003 (testing strategy), ADR-0004
(compatibility contract / supported servers), ADR-0005 (pull-first results),
ADR-0007 (exceptions), ADR-0012 (prepared statements and query modes).

## Tech stack and requirements

- **Language:** plain Java. No Kotlin, Scala, or other JVM languages.
- **Java 21+.** Prefer records, sealed types, pattern matching, `var`, text
  blocks, enhanced switch where they improve clarity.
- **Virtual threads are first-class.** The I/O layer is blocking-style code run
  on virtual threads, not an async/event-loop framework. Never hold a
  `synchronized` monitor across blocking I/O (use `ReentrantLock`) or virtual
  threads pin to carrier threads.
- **Build:** Apache Maven; `./mvnw` wrapper is checked in. A plain `mvn` must work.
- **Dependencies:** minimal, ideally zero at runtime for the core driver.
  `postgresql-client-protocol` stays dependency-free. Test-only and build-time deps are fine.
  Adding any runtime dependency is a decision to raise explicitly, not to make
  silently (see `docs/dependencies.md`, ADR-0002).

## Module layout

Dependencies flow strictly one way:
`postgresql-client-pgjdbc-compat` -> `postgresql-client-jdbc` -> `postgresql-client` -> `postgresql-client-protocol`.
No reverse or cyclic edges; no `java.sql.*` below `postgresql-client-jdbc`.

- **`postgresql-client-protocol`** - wire protocol encode/decode only. Pure serialization;
  no sockets, connection state, or I/O policy.
- **`postgresql-client`** - the driver: connections, auth (incl. SCRAM), TLS, simple
  and extended query protocols, COPY, LISTEN/NOTIFY, the native public API.
- **`postgresql-client-jdbc`** - the `java.sql.*` adapter on top of core.
- **`postgresql-client-pgjdbc-compat`** - `org.postgresql.*` source-compatibility layer on
  top of the JDBC module. A migration aid, not yet a certified drop-in.
- **`postgresql-client-bench`** (pgbench-style JDBC benchmark), **`postgresql-client-bench-jmh`**
  (server-free micro-benchmarks), **`postgresql-client-coverage`** (aggregate JaCoCo),
  **`postgresql-client-native-smoke`** (GraalVM native-image metadata gate). All
  build-only; never published.

Each shipped module has a `module-info.java`. **A new public package must be
exported there**, or downstream modules fail to compile on the module path.

## Build and test

```sh
./mvnw clean install                 # full build + unit tests + gates
./mvnw verify                        # what CI's unit job runs
./mvnw test                          # unit tests only (no gates, no Docker)
```

Fast inner loops (use these; a full reactor build is rarely what you want):

```sh
./mvnw -q -pl postgresql-client -am test -Dtest=PullResultStreamTest
./mvnw -q -pl postgresql-client -am test -Dtest='Numeric*Test#roundTrips*'
./mvnw -q -pl postgresql-client-jdbc -am -o test    # -o offline once deps are cached
```

- `-pl <module> -am` builds only that module and its upstreams.
- Add `-Dsurefire.failIfNoSpecifiedTests=false` when `-Dtest` targets a class
  that does not exist in every reactor module you built.
- `-T 1C` reactor parallelism is on by default via `.mvn/maven.config`; force a
  serial build with `-T 1` when debugging interleaved output.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pgjdbc/pg-java](https://github.com/pgjdbc/pg-java) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
