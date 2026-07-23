---
trigger: always_on
description: Kotest is a Kotlin Multiplatform testing framework comprising three pillars: a test framework with 9+ spec styles, an assertion/matchers library, and a property-based testing engine. It targets JVM, JS, WasmJS, WasmWASI, and all major Native platforms (Linux, macOS, Windows, iOS, tvOS, watchOS).
---

# Kotest - Working with this Codebase

## What is Kotest

Kotest is a Kotlin Multiplatform testing framework comprising three pillars: a test framework with 9+ spec styles, an assertion/matchers library, and a property-based testing engine. It targets JVM, JS, WasmJS, WasmWASI, and all major Native platforms (Linux, macOS, Windows, iOS, tvOS, watchOS).

## Build and Test

```bash
# Full check (all platforms enabled locally by default)
./gradlew check

# JVM-only check (much faster for most changes)
./gradlew check -PjvmOnly=true

# Single module
./gradlew :kotest-assertions:kotest-assertions-core:check
./gradlew :kotest-framework:kotest-framework-engine:jvmTest

# API compatibility check (required before PR)
./gradlew apiCheck

# Regenerate API dump after public API changes
./gradlew apiDump
```

Gradle properties `kotest_enableKotlinJs` and `kotest_enableKotlinNative` in `gradle.properties` control which targets are built. Set to `false` for faster local iteration when working on JVM-only code.

The project requires JVM heap of 8GB (`-Xmx8g` in `gradle.properties`). Gradle parallel execution and caching are enabled by default.

## Project Structure

```
kotest-common/                          # Shared annotations (@ExperimentalKotest, @KotestInternal, etc.)
kotest-framework/
  kotest-framework-engine/              # Core engine: spec styles, lifecycle, config, extensions
  kotest-framework-standalone/          # Fat jar for standalone execution
  kotest-framework-plugin-gradle/       # Gradle plugin for running tests outside Gradle's test task
  kotest-framework-symbol-processor/    # KSP processor for generating KMP test entries
kotest-assertions/
  kotest-assertions-shared/             # Matcher<T>, MatcherResult, assertSoftly, error collectors
  kotest-assertions-core/              # All standard matchers + nondeterministic helpers (eventually, continually)
  kotest-assertions-json/              # JSON matchers
  kotest-assertions-ktor/             # Ktor HTTP matchers
  kotest-assertions-arrow/            # Arrow matchers
  kotest-assertions-compiler/         # Compile-time assertion matchers
  kotest-assertions-table/            # Legacy data/table testing from kotest 4.x
  kotest-assertions-konform/          # Konform validation matchers
  kotest-assertions-kotlinx-datetime/ # kotlinx-datetime matchers
  kotest-assertions-yaml/            # YAML matchers
kotest-property/                       # Property testing: Gen, Arb, Exhaustive, shrinking, seed persistence
  kotest-property-lifecycle/          # Property testing + framework lifecycle integration
  kotest-property-permutations/       # 6.0+ permutations DSL
  kotest-property-arrow/              # Arrow generators
  kotest-property-datetime/           # kotlinx-datetime generators
kotest-extensions/                     # Third-party integrations (Spring, Testcontainers, Allure, Koin, etc.)
kotest-runner/
  kotest-runner-junit-platform/       # Shared JUnit Platform support
  kotest-runner-junit5/               # Primary JVM runner (Gradle/IntelliJ integration)
  kotest-runner-junit4/               # Android instrumented test runner
  kotest-runner-junit6/               # JUnit6 runner
kotest-intellij-plugin/               # IntelliJ IDEA plugin
kotest-bom/                           # Bill of Materials
kotest-tests/                          # Integration/regression test suites (each is a separate Gradle module)
buildSrc/                             # Convention plugins and CI configuration
```

## Coding Conventions

- Follow [Kotlin Coding Conventions](https://kotlinlang.org/docs/reference/coding-conventions.html) with **3-space indentation** (not 4).
- Minimize mutability.
- Choose self-explanatory names.
- Source sets follow KMP layout: `commonMain`, `jvmMain`, `jsMain`, `nativeMain`, `commonTest`, `jvmTest`, etc.
- The base package is `io.kotest`. Test code historically also uses `com.sksamuel.kotest` (legacy package from pre-rename era).

## Key Architectural Concepts

### Spec Styles
Each spec style (FunSpec, StringSpec, BehaviorSpec, DescribeSpec, ShouldSpec, WordSpec, FreeSpec, FeatureSpec, ExpectSpec) is an abstract class that extends `DslDrivenSpec -> Spec`. The DSL is injected via scope interfaces (e.g., `FunSpecRootScope`) which define the test registration methods. All styles are functionally equivalent -- they differ only in DSL syntax.

### Matcher Architecture
The `Matcher<T>` interface has a single `test(value: T): MatcherResult` method. Results come in three variants:
- `SimpleMatcherResult` -- basic pass/fail with message lambdas
- `DiffableMatcherResult` -- includes actual/expected for IntelliJ diff links
- `ThrowableMatcherResult` -- carries a pre-built exception

Build results using `MatcherResultBuilder`. Matchers compose via `and`/`or` infix functions and `contramap` for type adaptation.

### Extension vs Listener

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kotest/kotest](https://github.com/kotest/kotest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
