---
trigger: always_on
description: SPDX-License-Identifier: Apache-2.0
---

<!--
SPDX-License-Identifier: Apache-2.0

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

# Agent Guide for grails-intellij-plugin

> **IMPORTANT**: This is the IntelliJ IDEA plugin for Apache Grails (GSP language support,
> Grails project structure and navigation, run configurations, taglib/domain-class
> support), NOT the Grails framework itself and NOT a Grails application.

## Quick Reference

```bash
# Compile and run all tests (~3.5 min, ~970 tests)
./gradlew test

# Single test class / single test method
./gradlew test --tests "org.apache.grails.intellij.plugin.SomeTest"
./gradlew test --tests "org.apache.grails.intellij.plugin.SomeTest.testFeature"

# Build the plugin ZIP (written to build/distributions/)
./gradlew buildPlugin

# Full check / Plugin Verifier / license audit / sandbox IDE
./gradlew check
./gradlew verifyPlugin
./gradlew rat
./gradlew runIde
```

## Critical Rules

1. **Never trust a test run without checking for compile errors first.** If compilation
   fails during `./gradlew test`, Gradle silently runs the **stale previous bytecode**.
   Always grep the output for `error:` before believing pass/fail results or probe output.
2. **Collect failure lists from a full run only.** A `--tests`-filtered run **wipes**
   `plugin/build/test-results/test/`, destroying the results of the previous full run.
3. **Every code change must include tests.** Review the existing tests for the affected
   area before making changes, and keep every test that covers a modified class in sync
   with the new behavior. Run all affected tests and ensure they pass before committing.
4. **Apache license header required** on all new source files (enforced by `./gradlew rat`).
   Every RAT exclude in the `rat` convention plugin must carry an inline justification.
   - **New files use the ASF header** (verbatim from the `HEADER` file: *"Licensed to the
     Apache Software Foundation (ASF) under one or more contributor license agreements…"*),
     rendered as a `/* … */` block for Java/Kotlin or `<!-- … -->` for XML. **Do NOT** copy
     the `Copyright 2000-2026 JetBrains s.r.o. and contributors` header — that appears only on
     files inherited through the migration; new work is ASF-owned.
5. **Test-fixture JDK conventions (2026.2+)** — Mock JDK 1.7 is no longer shipped:
   - **Light fixtures**: `GrailsTestCase` pins `DefaultLightProjectDescriptor(IdeaTestUtil::getMockJdk11)`
     via `getTestJdk()`. Override `getTestJdk()` to a real JDK only when the test needs
     Swing/AWT classes.
   - **Heavy fixtures** (`JavaModuleFixtureBuilder`): use
     `moduleBuilder.addJdk(System.getProperty("java.home"))`. `IdeaTestUtil.getMockJdk11()`
     NPEs in `tuneFixture` because the test application doesn't exist yet.
   - Avoid `RepositoryTestLibrary`-backed descriptors (`GroovyProjectDescriptors.GROOVY_*`)
     — they fail light-project init in the plugin-dev SDK ("Cannot find IntelliJ IDEA
     project files"). Prefer `GroovyProjectDescriptors.MOCK_JDK_11` unless the test truly
     needs a Groovy jar on the classpath.
6. **Don't add license headers to `testdata/`** — the content *is* the test input;
   headers break parser/position-sensitive tests.
7. **No wildcard imports** — use explicit imports, matching the existing sources.
8. **JDK is pinned via `.sdkmanrc`** (Java 25, Gradle 9.6.1) — no Gradle toolchain on
   purpose, for reproducible builds. Run `sdk env` if the build complains about the JDK.
9. **Remove debug probes before committing** (see Debugging below).
10. **Retry transient commit failures.** Git commits can fail with
    `1Password: failed to fill whole buffer` (signing) — just retry.

## Technology Stack

| Component | Version |
|-----------|---------|
| IntelliJ Platform | 2026.2 Ultimate (`sinceBuild` 262) |
| JDK (build) | 25 (pinned in `.sdkmanrc`) |
| JDK (`grails-rt`, `grails-compiler-patch`, `jps-plugin`) | targets Java 8/11 |
| Gradle | 9.6.1 (wrapper) |
| IntelliJ Platform Gradle Plugin | 2.x |
| Kotlin | 2.4.x (stdlib not bundled) |
| Tests | JUnit 4 + AssertJ + IntelliJ test framework (light/heavy fixtures) |

## Project Structure

A composed build: `build-logic` is an included build holding every convention plugin, and the
subprojects sit in tier directories. Project names carry the subpath, so
`pluginModules/hibernate` is the Gradle project `:pluginModules-hibernate`. The root project
is a pure aggregator — it owns only RAT and coverage aggregation, no sources.

| Path | Gradle project | Description |
|------|----------------|-------------|
| `plugin/` | `:plugin` | Main plugin: GSP language, Grails project support, run configs |
| `pluginModules/{copyright,coverage,hibernate,i18n,langInjection,maven}/` | `:pluginModules-*` | Optional IntelliJ content modules (`pluginModule` deps) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/grails-intellij-plugin](https://github.com/apache/grails-intellij-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
