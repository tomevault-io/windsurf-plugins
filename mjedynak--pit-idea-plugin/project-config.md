---
trigger: always_on
description: **After every code change, update this file to reflect the new state of the project.**
---

# AGENTS.md — PIT Idea Plugin

**After every code change, update this file to reflect the new state of the project.**

## Project Overview

IntelliJ IDEA plugin for [PIT Mutation Testing](http://pitest.org). Adds a run configuration and context menu actions to execute PIT directly within the IDE. After a run finishes, mutation lines are marked in the editor with color-coded gutter bands mirroring the PIT HTML report: light green for covered lines (mutations `KILLED`/`NON_VIABLE`), light red for uncovered lines (any `SURVIVED`/`NO_COVERAGE`), blue-grey for other statuses; each band carries the mutation descriptions as a hover tooltip (gutter icon + scrollbar stripe) and a right-click context menu to clear the markings. Current version: **1.4.13-SNAPSHOT**, bundling PIT **1.30.0** and JUnit5 plugin **1.2.3**.

## Build & Run Commands

```bash
./gradlew build          # Full build (compile + test + format check)
./gradlew test           # Run unit tests only
./gradlew integrationTest # Run integration tests (starts real IDE with plugin)
./gradlew spotlessApply  # Auto-format code (Java + Kotlin)
./gradlew spotlessCheck  # Check formatting without modifying
```

CI runs `./gradlew integrationTest build` on push/PR to `master` (Java 25 temurin, Ubuntu).

**After every significant code change, run `./gradlew integrationTest` to verify the plugin loads and runs PIT end-to-end in a real IDE process.** `build` alone does not cover this.

## Project Structure

```
src/main/kotlin/pl/mjedynak/idea/plugins/pit/
├── actions/          # IntelliJ context menu actions (PitAction hierarchy)
│   ├── PitAction.kt
│   ├── PitActionUtils.kt
│   ├── DirectoryOrFilePitAction.kt
│   ├── RunAllPitAction.kt
│   ├── RunSomeTestsPitAction.kt
│   └── PitTestSomeClassesAction.kt
├── cli/              # CLI argument model and container
│   ├── PitCommandLineArgumentsContainer.kt
│   ├── PitCommandLineArgumentsContainerImpl.kt
│   ├── model/
│   │   └── PitCommandLineArgument.kt
│   └── factory/
│       ├── DefaultArgumentsContainerFactory.kt
│       └── DefaultArgumentsContainerPopulator.kt
├── configuration/    # Run configuration (PitRunConfiguration, PitConfigurationType)
│   ├── PitRunConfiguration.kt
│   ├── PitRunConfigurationFactory.kt
│   ├── PitConfigurationType.kt
│   └── PitRunConfigurationStorer.kt
├── gui/              # Settings editor form + populators
│   ├── PitConfigurationForm.kt
│   └── populator/
│       ├── PitConfigurationFormPopulator.kt
│       └── ProgramParametersListPopulator.kt
├── JavaParametersCreator.kt       # Builds JavaParameters for PIT execution
├── ClassPathPopulator.kt          # Assembles PIT classpath from plugin dir
├── editor/             # Editor coverage annotation after PIT run
│   ├── MutationStatus.kt            # PIT mutation status enum (KILLED/SURVIVED/NO_COVERAGE/...) + fromXml
│   ├── MutationReportParser.kt      # Parses mutations.xml; keeps ALL mutations incl. NO_COVERAGE
│   ├── CoverageLineMarkerRenderer.kt # Status-colored gutter band (LineMarkerRendererEx, Position.LEFT)
│   └── PitCoverageAnnotator.kt       # Project service: parses report + annotates open editors (band + gutter icon w/ hover tooltip + right-click clear menu)
├── console/DirectoryReader.kt     # Finds latest report directory
├── gradle/GradleProjectDeterminer.kt           # Detects Gradle projects
└── maven/                        # Maven project detection + pom.xml parsing
    ├── MavenProjectDeterminer.kt
    └── MavenPomReader.kt

src/testSupport/kotlin/pl/mjedynak/idea/plugins/pit/
├── PitTestHelper.kt           # E2E test helper (creates PitRunConfiguration and executes it)
├── PitActionTestHelper.kt     # Verifies action visibility/enablement and simulates user clicking the action
├── PitOutputReader.kt         # Reads PIT process info via reflection
└── PitCoverageTestHelper.kt   # Opens Calculator.java, polls editors for CoverageLineMarkerRenderer, returns line:STATUS:HAS_TOOLTIP triplets
src/test/kotlin/                   # All tests are Kotlin (9 test files)
src/integrationTest/
├── kotlin/                        # Integration tests using IntelliJ Starter framework
│   └── PitPluginIntegrationTest.kt  # 3 test scenarios: (1) plugin load + config + PIT via helper, (2) action visibility check + simulated click + PIT report, (3) coverage markers mirror report (6/10 COVERED, 14 UNCOVERED)
└── resources/testProject/         # Test project with Calculator.java, CalculatorTest.java
META-INF/plugin.xml                # Plugin descriptor (actions, extensions)
```

## Technology Stack

- **Languages**: Kotlin (all plugin source code)
- **Build**: Gradle 9.6.1, Kotlin DSL, IntelliJ Platform Gradle Plugin 2.18.1
- **Target**: IntelliJ IDEA 2026.2, Java 25 toolchain
- **PIT**: 1.30.0 (bundled as non-transitive dependencies)
- **Testing**: JUnit 5 (Jupiter 6.1.1) + Mockito-Kotlin 6.3.0
- **Formatting**: Spotless — ktlint (Kotlin)

## Code Conventions

- **Formatting is enforced** — run `./gradlew spotlessApply` before committing
- **Test method names** use backtick-quoted descriptive names: `` `should return absent for empty directory` ``
- **Nullable IntelliJ APIs** handled with `?.` safe calls in Kotlin

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mjedynak/pit-idea-plugin](https://github.com/mjedynak/pit-idea-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
