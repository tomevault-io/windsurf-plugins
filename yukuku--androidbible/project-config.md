---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents when working with code in this repository.

## Project Overview

**Bible for Android** (Alkitab / Quick Bible) — a 100% free, open-source Bible reader app for Android. Published on Google Play as "Alkitab" (Indonesian) and "Quick Bible" (non-Indonesian). The codebase supports multiple product flavors, 100+ downloadable Bible versions, song books, devotions, reading plans, cloud sync, and a daily verse widget.

- Official site: https://alkitab.app
- Developer docs: https://alkitab.app/developer

## Branches

**`develop` is the main integration branch — base all work on `develop` and target every PR at `develop`, never `master`.** The repo follows git-flow: `master` only receives merges from `release/*` branches when a version ships.

## Complete implementation tasks with a pull request

Unless the user explicitly requests otherwise, finish every implementation task by committing the changes, pushing a task branch to `yukuku/androidbible`, and opening a pull request targeting `develop`. This is part of completing the task, including requests phrased as building a feature or fixing a bug; do not ask for separate permission to create the PR.

Use a branch in this repository so the PR triggers GitHub Actions to build signed release APKs and publish APK preview links. See [the CI and preview documentation](docs/build-system.md#cicd).

- Run the checks appropriate to the changes before pushing; CI supplements local verification.
- When continuing a task that already has an open PR, push updates to its branch and update its description as needed.
- Return the PR link and the observed CI status to the user.
- Leave merging the PR and publishing a production release to an explicit user request.

Read-only analysis and advice do not require a PR.

## Build Commands

```bash
# Debug APK (open-source build, works out of the box)
./gradlew assemblePlainDebug

# Debug App Bundle
./gradlew bundlePlainDebug

# Unit tests (same as CI)
./gradlew testPlainDebugUnitTest testPlainReleaseUnitTest

# Run a single test class
./gradlew testPlainDebugUnitTest --tests "yuku.alkitab.base.util.QueryTokenizerTest"

# Run a single test method
./gradlew testPlainDebugUnitTest --tests "yuku.alkitab.base.util.QueryTokenizerTest.testQuotedPhrases"
```

**Requirements**: JDK 21 (Zulu recommended), Android SDK with compile SDK 36, NDK 28.2.13676358.

**Versioning**: `versionName` comes from `version.properties` (`versionBase` +
`betaNumber`) combined with a stage: `dev` by default (`5.0.0-dev.<commits
since versionBase last changed>`), `beta` via `-PversionStage=beta`
(`5.0.0-beta.<n>`), `release` via `-PversionStage=release` (`5.0.0`). `versionCode` is derived from wall-clock
time and is unrelated to the stage. Never hardcode a version in
`Alkitab/build.gradle.kts`; see the "Versioning" section of
`docs/build-system.md`.

The `plain` flavor is the open-source development build and works out of the box with the placeholder `Alkitab/google-services.json` checked into the repo (Firebase features won't function at runtime, but the app builds and runs). Production flavors (`yuku_alkitab`, `yuku_quick_bible`, `sabda_alkitab`) require:
- `$ALKITAB_PROPRIETARY_DIR/overlay/<applicationId>/text_raw/` — proprietary Bible text
- `$ALKITAB_PROPRIETARY_DIR/google-services.json` — real Firebase config (one file with client entries for all production applicationIds)
- Signing-key env vars: `SIGN_KEYSTORE`, `SIGN_ALIAS`, `SIGN_PASSWORD`. The `sabda_alkitab` flavor has its own upload key and needs `SIGN_SABDA_KEYSTORE`, `SIGN_SABDA_ALIAS`, `SIGN_SABDA_PASSWORD` instead (there is no fallback to `SIGN_*`)

With those set, build with a plain `./gradlew assembleYuku_alkitabRelease` (or any other production flavor).

### Building in the Claude Code sandbox (one-time setup)

The Claude Code VM does not ship with the Android SDK — you must provision it yourself before running Gradle, and do not rely on GitHub Actions for verification. Its preinstalled JDK is version 21, matching the `jvmToolchain(21)` used across modules; run `java -version` first, and skip step 1 below if it already reports 21. Prefer `plainDebug` since it needs no proprietary overlay or signing secrets.

Install paths used below (pick any, but keep them consistent):

- JDK 21 (only if not already preinstalled): `/home/user/tools/zulu21.52.203-ca-jdk21.0.12.1-linux_x64`
- Android SDK: `/home/user/android-sdk`

**Disk footprint:** expect ~5 GB across all of these combined (less if the preinstalled JDK 21 is used directly) — NDK r28c alone is ~2 GB unpacked, the rest of the SDK is ~1 GB, and `~/.gradle` grows to ~2 GB after the first build. Check free space before starting.

**Environment.** Every step below needs the same env vars. Write them once and source them each time — point `JAVA_HOME` at the preinstalled JDK (`dirname $(dirname $(readlink -f $(which java)))`) unless you installed Zulu JDK 21 yourself in step 1, in which case use that path instead:

```bash
cat > /home/user/tools/android-env.sh <<'EOF'
export JAVA_HOME="$(dirname "$(dirname "$(readlink -f "$(which java)")")")"
export ANDROID_HOME=/home/user/android-sdk

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yukuku/androidbible](https://github.com/yukuku/androidbible) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
