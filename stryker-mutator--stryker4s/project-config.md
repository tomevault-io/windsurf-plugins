---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Stryker4s is a mutation testing framework for Scala. It mutates source files, recompiles once, and runs the test suite per mutant to see which mutations survive. It ships as four build-tool integrations (sbt, Mill, Maven, standalone command runner) over a shared `core`.

## Commands

The build is **sbt 2.0.x**. Prefer `sbt --client` to reuse the running server. In VS Code, prefer the Metals MCP tools (`compile-module`, `test`, `format-file`) over shelling out.

```bash
# Compile + unit tests, matching CI
sbt 'compile; ...@scalaBinaryVersion=3/test'

# Single suite, then narrowing within it (munit inherits junit-interface's filter args)
sbt 'core/testOnly stryker4s.mutants.MutatorTest'
sbt 'core/testOnly stryker4s.mutants.MutatorTest -- *some glob*'
sbt 'core/testOnly stryker4s.mutants.MutatorTest -- "--tests=.*regex.*"'

# Formatting (scalafmt is run via the wrapper script, not an sbt plugin)
./bin/scalafmt
./bin/scalafmt --test    # CI check

# Integration tests (each publishes the artifacts it needs first)
sbt 'sbtPlugin/scripted; sbtPlugin3/scripted'   # modules/sbt/src/sbt-test/
sbt 'millPlugin/millScripted'                    # modules/mill/src/mill-test/
sbt publishM2Local && cd maven && mvn verify     # Maven plugin

# Dogfooding: run Stryker4s on itself
sbt 'core/stryker'          # also commandRunner/stryker, millPlugin/stryker
```

Local-publish aliases (defined in `build.sbt`) for testing a plugin against a real project: `publishPluginLocal`, `publishMillLocal`, `publishCommandRunnerLocal` (all `0.0.0-TEST-SNAPSHOT`), `publishM2Local` (`SET-BY-SBT-SNAPSHOT`, for Maven).

Commits follow Conventional Commits — release-please derives releases from them.

## Module graph

```
api ──────────────┐              (Logger/Level only, no deps)
testRunnerApi ────┼── core ── commandRunner
   (protobuf)     │     ├───── sbtPlugin / sbtPlugin3
testRunner ───────┘     ├───── millPlugin
                        └───── maven/ (separate Maven build)
testkit ── test-only munit base suites, used by core/sbt/mill/commandRunner
```

- **`core`** — everything build-tool agnostic: config, file resolution, mutation, run orchestration, reporting.
- **`api`**, **`testRunnerApi`**, **`testRunner`** — cross-built to Scala 2.12/2.13/3.3-LTS because they are injected onto the *user's* classpath. Keep them dependency-light and LTS-compatible.
- **`maven/`** is not part of the sbt build. It is a Maven project that consumes `stryker4s-core` from `~/.m2`; run `sbt publishM2Local` before touching it.

### Scala versions (`project/Dependencies.scala`)

`core`/`testkit` cross-build to Scala 3 and **2.12** (needed by the sbt 1.x plugin). Consequence: **core source must compile under Scala 2.12 with `-Xsource:3`**, not real Scala 3. Use `implicit`, not `given`; wildcard `*` imports and `?` are fine. Version-specific code goes in `src/main/scala-2.13-` / `scala-2.13+`. The unqualified sbt project id (e.g. `core`) is the Scala 3 variant; `core2_12` is the 2.12 one.

The sbt plugin has two variants: `sbtPlugin` (Scala 2.12 / sbt 1.x) and `sbtPlugin3` (Scala 3 / sbt 2.x), with a `PluginCompat` shim per version under `modules/sbt/src/main/scala-2.12` and `scala-3`. The Mill plugin is pinned to the Scala version of the minimum supported Mill.

## Architecture

### Mutation switching

The central technique (see [docs/contributing.md](docs/contributing.md)). Rather than compiling once per mutant, all mutants for a statement are compiled into a single pattern match and selected at runtime:

```scala
_root_.stryker4s.activeMutation match {
  case 1 => person.age > 18
  case _ =>
    _root_.stryker4s.coverage.coverMutant(1)  // coverage collection
    person.age >= 18                           // original
}
```

This means generated code must always compile. Two consequences run through the codebase: the sbt runner strips fatal-warning/unused scalac options from the target project, and mutants that *do* cause compile errors are located and rolled back (`RollbackHandler`, `MutantInstrumenter.mutantIdsForCompileErrors`) before a retry run.

### Pipeline

`Stryker4sRunner` (abstract, one impl per build tool) wires the graph and runs `Stryker4s.run()`:

1. **`FileResolver`** (`GlobFileResolver`) → stream of files matching the `mutate` globs.
2. **`Mutator`** ([modules/core/.../mutants/Mutator.scala](modules/core/src/main/scala/stryker4s/mutants/Mutator.scala)) — an fs2 pipeline: parse with scalameta (`MutantFinder`) → `MutantCollector` walks the tree (`TreeTraverser`) and `MutantMatcher` produces candidate mutations → assign ids → split ignored vs. active → `MutantInstrumenter` rewrites each file into mutation switches. Parallelism is `Config.cpuParallelism`.
3. **`MutantRunner`** — copies the project into a temp dir under `target/` with mutated files substituted, builds a `TestRunnerPool`, does an **initial test run** (must pass; also collects coverage + a timeout baseline), then runs each covered mutant. Mutants that are static or uncovered are short-circuited to `Ignored`/`NoCoverage` without running tests.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stryker-mutator/stryker4s](https://github.com/stryker-mutator/stryker4s) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
