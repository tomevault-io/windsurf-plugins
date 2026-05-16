---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project context

ConsultMe is a **Jetpack Compose template** for new Android apps — multi-module, Kotlin-only, with code-quality plumbing (Spotless + ktlint, Android Lint) wired in. Apps generated from this template start by replacing the placeholder content in `:feature-example` (see README.md "How to Rename and Refactor"). Default package is `com.thecompany.consultme` and is expected to be renamed downstream.

**Roadmap and ongoing improvements** live in `docs/IMPROVEMENT_PLAN.md`. Check it before starting non-trivial work — it lists what's intentionally deferred (AGP 9, Hilt 2.59+, Kotlin 2.3.20) and what the next planned phases are.

## Common commands

CI runs these in order; locally you typically want the same gates before opening a PR:

```bash
./gradlew spotlessCheck     # formatting (fails on missing license header / ktlint violations)
./gradlew spotlessApply     # autofix
./gradlew lintRelease       # Android lint, release variant
./gradlew test              # all unit tests
./gradlew koverHtmlReport   # aggregated coverage at build/reports/kover/html/
./gradlew moduleGraph       # regenerate docs/MODULE_GRAPH.md (CI fails if stale)
./gradlew connectedAndroidTest  # instrumented tests (needs device/emulator)
```

Running one test:
```bash
./gradlew :feature-example:testDebugUnitTest --tests "com.thecompany.consultme.feature.example.ui.ExampleUnitTest"
./gradlew :feature-example:testDebugUnitTest --tests "*ExampleUnitTest.someMethod"
```

`spotlessApply` is the autofix; the license header gets injected with the current year and the value of `template.company` from `gradle.properties` (defaults to `MyCompany`). Adopters override `template.company` once per fork; the bootstrap script (`scripts/rename-template.py`) handles this automatically.

Lint baselines (`<module>/lint-baseline.xml`) exist per module — regenerate with `./gradlew :<module>:updateLintBaseline` rather than hand-editing.

## Module graph

- `:app` — Application module. Wires Hilt (`ConsultMeApplication`), Compose root, navigation. Depends on `:core-designsystem`, `:core-ui`, `:feature-example`. Consumes the baseline profile produced by `:baselineprofile` (via `androidx.baselineprofile` + `androidx.profileinstaller`).
- `:baselineprofile` — `com.android.test` producer that generates `app/src/main/baseline-prof.txt`. Houses `BaselineProfileGenerator` (one-shot collect via `BaselineProfileRule`) and `StartupBenchmarks` (cold-start macrobenchmark, two compilation modes). Built with `consultme.android.baselineprofile`. Regenerate the profile via `./gradlew :app:generateReleaseBaselineProfile`.
- `:feature-*` — Screen-level features (currently only `:feature-example`, a placeholder for adopters to replace; rename it once you know what you're building). Get `:core-designsystem` + `:core-ui` + `:core-testing` automatically via the `consultme.android.feature` convention.
- `:core-designsystem` — Compose theme + design tokens (colors, typography). Owns `ConsultMeTheme`. Adopters extend with icon registries, custom components, etc.
- `:core-ui` — Shared Compose composables not part of the design system (loading/empty/error states). Currently a scaffold.
- `:core-model` — Pure-Kotlin data classes (no Android, no Hilt). Built with `consultme.jvm.library`.
- `:core-common` — Pure-Kotlin shared utilities; ships the NIA-style `Dispatcher` qualifier + `AppDispatchers` enum. Built with `consultme.jvm.library`.
- `:core-domain` — Pure-Kotlin use-cases. Depends on `:core-model`. Built with `consultme.jvm.library`.
- `:core-data` → `:core-database` — Repository and persistence layers.
- `:core-testing` — **Test fixtures shared across every module.** Uses `api(...)` (not `implementation`) to re-export JUnit, Truth, Turbine, MockK, Hilt testing, coroutines-test, Espresso. It also provides `HiltTestRunner`, which every module references as its `testInstrumentationRunner`. Consume it via `testImplementation(project(":core-testing"))` and `androidTestImplementation(project(":core-testing"))` — do **not** add JUnit/Espresso/etc. directly in module build scripts.

## Conventions enforced by tooling

- **License header**: Spotless (configured in root `build.gradle.kts`) requires `// Copyright $YEAR MyCompany` on every `.kt` and `.gradle.kts` file. Placement is delimiter-driven: above the `package` line for Kotlin, above the first `/*` for Gradle Kotlin scripts. New files without the header fail `spotlessCheck`. The "MyCompany" / `$YEAR` literals get rewritten by `spotlessApply`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tarek-Bohdima/ConsultMe](https://github.com/Tarek-Bohdima/ConsultMe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
