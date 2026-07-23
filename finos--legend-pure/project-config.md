---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Legend Pure is the language and compiler engine powering the FINOS Legend data-management platform. This repo ships the Pure language, its compiler, two execution engines (compiled + interpreted), DSL extensions, a relational store, and the Maven plugin suite that drives Pure compilation during normal Maven builds.

Group ID: `org.finos.legend.pure`. Current version: `5.81.1-SNAPSHOT` (root `pom.xml`). Downstream consumer: `legend-engine` compiles against these JARs, so public Java API in `legend-pure-m4`, `legend-pure-m3-core`, and runtime engine modules is a **breaking-change surface for the whole Legend stack**.

## Build & test commands

Requires **JDK 11 or 17** (enforcer rejects anything else) and Maven 3.6+.

```bash
# First build, skip tests (15–30 min warm repo, longer cold)
mvn -T 4 clean install -DskipTests

# Full build + tests
mvn clean install

# Single module
mvn test -pl legend-pure-core/legend-pure-m3-core

# Single test class / method (remember -DfailIfNoTests=false — most modules don't
# have a match and will fail without it)
mvn test -pl legend-pure-core/legend-pure-m3-core \
    -Dtest=TestM3Compiler -DfailIfNoTests=false
mvn test -pl legend-pure-core/legend-pure-m3-core \
    -Dtest="TestM3Compiler#testSimpleClass" -DfailIfNoTests=false

# PCT tests only (pattern-matched)
mvn test -Dtest="*_PCT" -DfailIfNoTests=false

# Checkstyle without tests
mvn verify -DskipTests
mvn checkstyle:check
# Skip checkstyle in a hurry
mvn clean install -Dcheckstyle.skip=true

# Build a module and its dependencies
mvn clean install -pl legend-pure-store/legend-pure-store-relational -am
```

If you see `cannot find symbol: class CoreInstance...`, you ran `mvn test` without prior code generation. Run `mvn generate-sources` or `mvn install -DskipTests` first — ANTLR4 parsers and `CoreInstance` accessors live in `target/generated-sources/` and must exist before `maven-compiler-plugin` runs.

Heap: the compiler builds large graphs. For local builds, `export MAVEN_OPTS="-Xmx4g"`.

macOS JDK switch: `export JAVA_HOME=$(/usr/libexec/java_home -v 11) PATH="$JAVA_HOME/bin:$PATH"`. The enforcer rejects 25, 21, 12–16, 18+ — must be 11 or 17.

After editing a `.pure` file, the compiled engine's per-function generated Java under `target/generated-test-sources/` can go stale. Run `mvn clean install -DskipTests -pl <engine-module> -am` (or just delete `target/generated-test-sources/`) to regenerate before `mvn test`.

When passing `-Dtest=...` with `-am`, add `-Dsurefire.failIfNoSpecifiedTests=false` — dependency modules without matching tests otherwise fail the reactor.

## Architecture — the M4 → M3 → M2 → M1 stack

The metamodel-layer split is **the** key concept for navigating this codebase:

- **M4** (`legend-pure-m4`) — meta-metamodel. Defines what a "node" is (`CoreInstance` interface, serialization primitives). Rarely touched.
- **M3** (`legend-pure-m3-core`) — the Pure language itself: `Class`, `Function`, `Association`, parser (ANTLR4 `M3.g4` + `M3AntlrParser`), compiler passes, standard library.
- **M2** (`legend-pure-dsl/*`, `legend-pure-store/*`) — DSL extensions built *in Pure*: mapping, diagram, graph, path, store, tds, relational. Each DSL ships three sub-modules: `*-pure` (Pure source), `*-grammar` (ANTLR4 + Java visitor), `*-runtime-*-extension` (compiled/interpreted runtime hook).
- **M1** — user/business Pure code. Lives in consumer repos, not here.

Rule of thumb: editing `m4` = changing what a node *is*; editing `m3-core` = changing what `Class` or `Function` *means*; editing a DSL module = changing what `Mapping` or `Database` *means*.

### Two execution modes (kept in lockstep by PCT)

Both modes compile from the same `CoreInstance` graph produced by the M3 compiler:

- **Compiled** (`legend-pure-runtime-java-engine-compiled`) — ahead-of-time codegen to Java during Maven build. Production path.
- **Interpreted** (`legend-pure-runtime-java-engine-interpreted`) — tree-walking interpreter at runtime. Dev/IDE path.

**PCT (Platform Compatibility Testing)** is the integration contract that keeps these engines identical: Pure functions carrying the `<<PCT.function>>` stereotype (with `<<PCT.test>>` tests) run on *both* engines every build; divergent results fail CI. Tests follow `Test_<Mode>_<Suite>_PCT` naming. The current branch (`pct-refactor`) is migrating Java-side abstract test classes into native Pure PCT — recent commits named `test(grammar): PCT … migration` are deleting `AbstractTest*.java` under `legend-pure-m3-core/src/test/java/.../function/base/` and replacing them with Pure PCT functions in `legend-pure-m3-core/src/main/resources/platform/pure/`.

### Maven plugin pipeline (driven by `legend-pure-maven/*`)

Pure compilation runs as part of the normal Maven lifecycle, not a separate step:

1. `legend-pure-maven-generation-platform-java` (phase `compile`) — generates M3 `CoreInstance` Java accessors.
2. `legend-pure-maven-compiler` (`compile`) — parses `.pure` files to binary elements.
3. `legend-pure-maven-generation-par` — serializes compiled repos to PAR archives (the build/startup cache).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [finos/legend-pure](https://github.com/finos/legend-pure) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
