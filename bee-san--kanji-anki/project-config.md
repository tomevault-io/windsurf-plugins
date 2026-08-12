---
trigger: always_on
description: This repo is an Android app that syncs against AnkiDroid's flashcard content
---

# Agent Testing Runbook

This repo is an Android app that syncs against AnkiDroid's flashcard content
provider. Provider bugs can look fixed in unit tests while still failing on a
real AnkiDroid install, so release fixes must include a live AnkiDroid emulator
run when the change touches sync/provider behavior.

## Gradle Command Note

This repo includes a checked-in Gradle wrapper. Run Gradle tasks from the repo
root with `./gradlew`, for example `./gradlew :core:test`.

For Android tasks on this machine, prefer the prepared SDK under `/tmp` when no
`local.properties` file is present:

```sh
ANDROID_HOME=/tmp/android-sdk ANDROID_SDK_ROOT=/tmp/android-sdk \
  ./gradlew :app:testDebugUnitTest :app:compileDebugAndroidTestJavaWithJavac
```

## CI And Static Analysis Notes

For the full operator triage checklist, gate map, and release path
invariants, see `docs/ci-sonar-reliability-runbook.md`.

The normal local confidence gate is `./gradlew ciFast`. It runs deterministic
JVM tests, coverage reports/checks, app unit tests, Android instrumentation
compilation, lint, and Python asset tests. GitHub's default Android CI splits
that same deterministic surface across parallel jobs and aggregates them under
the `Fast confidence gate` check. Do not use SonarCloud or CodeQL as a
substitute for the normal Android CI workflow.

`./gradlew ciQuality` produces the deterministic bytecode and coverage inputs
used by SonarQube. `./gradlew ciRelease` runs the release confidence gate and
assembles the signed release APK when signing environment variables are set.
In CI, the deterministic test surface is enforced by `Android CI` itself: the
auto-release only triggers off a successful `Android CI` run, so the release
workflow's validate job just assembles and verifies the signed APK. Manual tag
pushes and `workflow_dispatch` releases (which can target commits no CI run
has vouched for) run the deterministic unit-test surface inline before
assembling. `ciRelease` is the local gate, not the CI release build command.

Beta releases are cut automatically: every successful `Android CI` run on a `main`
push triggers `android-release.yml` through a `workflow_run` trigger, which
computes the next numeric `vMAJOR.MINOR.PATCH` tag, verifies the APK at that CI
run's commit, and publishes it as a GitHub prerelease (creating the tag at
publish time). Numeric tags preserve update discovery for Kani 0.5.10 and
0.5.11; GitHub's prerelease flag, rather than a tag suffix, keeps stable users
on the latest stable release. The parser still accepts explicit SemVer
`vMAJOR.MINOR.PATCH-beta` tags for compatibility and deliberate manual beta
releases. The release path is deliberately self-contained: it
does not poll SonarQube/CodeQL check runs and it never runs emulator jobs.
Those were the top causes of blocked, flaky, and multi-hour releases; SonarQube
and CodeQL are advisory scans on `main`, and live AnkiDroid provider coverage
lives in the nightly/dispatch `android-instrumented.yml` workflow plus the
stricter local gate below. `tools/test_release_workflows.py` locks these
invariants in. Manual tag pushes and `workflow_dispatch` with an explicit
`release_tag` still work for deliberate stable versions.
Choose a stable version whose numeric core is newer than every published beta;
do not turn `vX.Y.Z-beta` into `vX.Y.Z`, because both map to the same Android
`versionCode`.

SonarCloud and CodeQL run on pushes to `main`, and can also be run manually.
CodeQL also has a scheduled weekly run. If you change either workflow, push it
and watch the first GitHub Actions run to completion; local Gradle success
alone is not enough to validate the service integration.

For SonarCloud, keep the hard target on correctness and maintainability: the
quality gate must pass, and code smells, bugs, vulnerabilities, and unreviewed
security hotspots should be driven to zero. Treat 100% coverage as an
optimization target only. Do not slow the normal CI path just to chase perfect
coverage; prefer the fast deterministic coverage path by default and reserve
full connected coverage for manual investigation or release-risk checks.

The deterministic AnkiDroid fixture workflow runs nightly and through
workflow-dispatch only; it is not part of the release path. It generates a
small sanitized Kiku collection in CI, installs pinned AnkiDroid in an
emulator, grants the real provider permission, and runs the live-provider
sync subset with `kanjiLiveAnkiDroid=true` and a small
`kanjiLiveMinimumNotes` value.

The local real-collection live gate remains stricter. Do not cut a release for
provider/sync changes unless the local copied user-collection AnkiDroid run
passes with the default 7,000-note threshold.

Useful commands:

```sh
gh run list --repo bee-san/kanji_anki --limit 10
gh run watch RUN_ID --repo bee-san/kanji_anki --exit-status
```

The CodeQL workflow uses manual Java/Kotlin analysis for this Android Gradle
project. Keep the CodeQL build step after `github/codeql-action/init` as a
forced clean compile:

```sh
./gradlew clean :fsrs-java:compileJava :core:compileJava :app:compileDebugJavaWithJavac --no-daemon --no-build-cache
```

Do not simplify that to a normal compile. Gradle can mark the compile tasks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bee-san/kanji_anki](https://github.com/bee-san/kanji_anki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
