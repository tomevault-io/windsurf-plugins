---
trigger: always_on
description: Guidance for AI coding agents (and human newcomers) working in this repository. Kept
---

# AGENTS.md

Guidance for AI coding agents (and human newcomers) working in this repository. Kept
tool-agnostic on purpose — see also `DEVELOPER.md` for the deep parser/lexer walkthrough,
`PORTING.md` for what each IntelliJ Platform bump broke and why, and `README.md` for the
user-facing feature list.

## What this is

**SnakeCharm** is an IntelliJ Platform plugin (Kotlin) that adds IDE support for the
[Snakemake](https://snakemake.readthedocs.io/) workflow language to PyCharm and other
IntelliJ-based IDEs. It is built **on top of the bundled Python plugin's PSI/API** — most of its
extension points are registered against `language="Python"` and it extends Python parsing rather
than defining a language from scratch.

## Build & test

The Gradle build uses the JDK toolchain `javaVersion` names in `gradle.properties`, and
`.java-version` in the repo root carries the same number — read it there rather than trusting any
number written in prose, because it moves with the platform and differs per branch. **Launch Gradle
itself on that JDK**, not merely as an available toolchain, because the window is bounded at both
ends: too new and the pinned Gradle crashes with a cryptic error (Gradle 8.x on JDK 24 fails with
`Type T not present`), too old and `instrumentCode` dies loading platform classes (2026.2 emits Java
25, so a JDK 21 daemon gets `UnsupportedClassVersionError: … class file version 69.0`). Set
`JAVA_HOME` before building from the CLI and **verify it** with `"$JAVA_HOME/bin/java" -version`: on
macOS `/usr/libexec/java_home -v <n>` treats `<n>` as a *minimum*, so it can hand back something
newer, exit 0, and leave you with one of those two errors and no hint why. Use a jenv/asdf/SDKMAN
path (`jenv prefix "$(cat .java-version)"`) or an explicit install path.

```shell
./gradlew buildPlugin      # -> build/distributions/snakecharm-*.zip
./gradlew test             # JUnit + Cucumber suite
./gradlew runIde           # sandbox IDE with the plugin installed
./gradlew verifyPlugin     # IntelliJ Plugin Verifier
```

The target IDE (`platformType`/`platformVersion` in `gradle.properties`) is downloaded
automatically on first build (hundreds of MB). `platformType = PC` is PyCharm Community, `PY` is
PyCharm Professional. Note that **2025.2 is the last standalone PyCharm Community release** — from
2025.3 on the unified PyCharm ships only under the `PY` artifact.

**Wrappers bundle:** `:buildWrappersBundle` reads `snakemakeWrappersRepoPath` (a local
[snakemake-wrappers](https://github.com/snakemake/snakemake-wrappers) checkout) and, when that
property is set, runs as part of `prepareSandbox`, so it sits in front of `buildPlugin` and
`runIde` — but **not** the test tasks,
which route through `prepareTestSandbox` and the separate test bundle below. That
property is commented out in `gradle.properties` by default, so a plain `buildPlugin` / `runIde`
yields a plugin without wrapper completion and the other wrapper-driven features; pass it explicitly
to include them: `./gradlew buildPlugin -PsnakemakeWrappersRepoPath=/path/to/snakemake-wrappers` (on
TeamCity it comes from the wrappers VCS root — see issue #571). The test-only bundle
(`:buildTestWrappersBundle`, what `test` actually consumes) defaults to `testData/wrappers_storage`
and needs no property.

`prepareSandbox` reaches that bundle through `from(named("buildWrappersBundle"))`, which works only
because the task declares the file as `outputs.file(...)` — `from(<file provider>)` carries no task
dependency, and dropping the dependency produces a wrapper-less plugin *silently*, which has
happened twice (#588, #591). Two things not to "tidy up" there, both of which have been tried and
reverted: the `from(...)` must stay **unconditional**, or `buildWrappersBundle` leaves the task graph
when `snakemakeWrappersRepoPath` is unset and its `onlyIf` — the only place the "no wrappers bundled"
warning is logged — never runs; and `outputs.upToDateWhen { false }` must stay, because declaring
the wrappers checkout as an input is what lets Gradle skip the crawler and ship a stale bundle. The
`onlyIf` deletes any bundle an earlier run left behind; that is what keeps a stale one out, not a
gate around the copy.

**CLI build memory:** if `:compileKotlin` dies with `OutOfMemoryError: GC overhead limit exceeded`,
give the Kotlin daemon more heap — append `-Pkotlin.daemon.jvmargs=-Xmx4g` (transforming some large
generated methods can exhaust the default heap).

### Running tests

Tests are **Cucumber/Gherkin** feature files under `src/test/resources/features/**`, executed
through a single JUnit runner, `AllCucumberFeaturesTest` (glue/step definitions in
`src/test/kotlin/features/glue/`). There is no per-feature test class.

- **Run one feature:** add a `@here` tag above its `Feature:` line (or above a single `Scenario:` /
  `Scenario Outline:`) and set `tags = "not @ignore and @here"` in `AllCucumberFeaturesTest.kt`;
  revert both afterwards. If PR #577 lands, the runner edit becomes unnecessary — `test` there
  forwards `CUCUMBER_TAGS='@here'` to cucumber's `cucumber.filter.tags`, which overrides the
  annotation. Worth the trouble either way: it turns a 25-minute suite into a ~60-second one.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JetBrains-Research/snakecharm](https://github.com/JetBrains-Research/snakecharm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
