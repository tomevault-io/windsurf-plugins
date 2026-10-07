---
trigger: always_on
description: The contract for any agent (or human) working in this repo. Read this first. If something here is wrong or missing, propose the fix as a separate `harness:` PR rather than working around it.
---

# CLAUDE.md

The contract for any agent (or human) working in this repo. Read this first. If something here is wrong or missing, propose the fix as a separate `harness:` PR rather than working around it.

## What this is

A chess game for Android written in Jetpack Compose. It is a personal experimentation project, so modern and even pre-release Jetpack APIs are welcome here.

- Single module: `:app`, package `com.bentrengrove.chess`.
- `engine/`: pure Kotlin game logic (`Board`, `Game`, `AI`). **No Compose or Android UI imports**, apart from the existing `@DrawableRes` piece artwork mapping. Everything here must be unit-testable on the JVM.
- `gamescreen/`: the game UI (`GameView`, `BoardView`, `GameStatusView`, `CapturedView`) and `GameViewModel`, which owns the `Game` state and undo/redo history.
- `titlescreen/`: the title screen and falling-pieces background.
- `ui/`: theme, colours, shapes, typography.
- `MainActivity.kt`: Navigation 3 back stack (`Screen` keys) and shared element transitions.

Toolchain: AGP 9.x, Kotlin 2.4, Compose BOM, compileSdk/targetSdk 37, minSdk 28, JDK 17. Dependency versions are declared inline in `app/build.gradle` and the root `build.gradle`; there is no version catalog.

To find the latest version of a dependency, use `gh api repos/<owner>/<repo>/releases/latest -q .tag_name` for GitHub-hosted projects, or fetch Maven metadata (`https://repo1.maven.org/maven2/<group/path>/<artifact>/maven-metadata.xml`, or `https://dl.google.com/android/maven2/<group/path>/group-index.xml` for AndroidX and Google libraries). Don't guess versions.

## The one command that matters

```sh
./gradlew preflight
```

This runs `spotlessCheck` (ktlint), `:app:lintDebug`, `:app:testDebugUnitTest` and `:app:verifyRoborazziDebug` (screenshot tests, see below). CI runs exactly this. **It must pass before you open or update a PR.** If it fails, fix the cause. Do not skip, suppress or delete checks to make it pass.

Other useful commands:

- `./gradlew spotlessApply` to auto-fix formatting.
- `./gradlew :app:testDebugUnitTest --tests 'com.bentrengrove.chess.engine.GameCastlingTest'` to run one test class.
- `./gradlew :app:assembleDebug` to build an APK for device checks.

## Screenshot tests

JVM screenshot tests use Roborazzi and Robolectric, so no emulator is needed. They live in `app/src/test/java/com/bentrengrove/chess/screenshots/`. Baselines are committed in `app/src/test/screenshots/` and named `<TestClass>.<method>.png`.

- `./gradlew :app:verifyRoborazziDebug` compares against the baselines. It is part of `preflight`.
- `./gradlew :app:recordRoborazziDebug` re-records the baselines.
- `./gradlew :app:compareRoborazziDebug` writes visual diffs to `app/build/outputs/roborazzi/` without failing. Use it to see what changed.

Rules:

- **Record baselines on Linux**: in CI, on the agent runner, or in a Linux container. Font rendering and anti-aliasing differ on macOS, so baselines recorded on a Mac fail verification in CI.
- **Intentional UI change:** re-record, commit the updated PNGs in the same PR, and say which baselines changed and why in the PR description. The reviewer looks at the images.
- **Unexpected diff:** treat it as a bug. Never re-record just to make `verifyRoborazziDebug` pass.
- **New screen or significant composable:** add a screenshot test with light and dark variants, following the existing tests.
- Tests pin `@Config(sdk = [34])`. Robolectric's SDK 35+ runtimes need JDK 21, and this project builds on JDK 17. Don't raise the SDK without moving the toolchain to JDK 21 first.
- Keep screenshot tests deterministic: no real clocks, randomness or network. The compose test rule pauses infinite animations, so don't restart them.

## Rules

### Code

- Match the surrounding style. ktlint is the source of truth for formatting, and `@Composable` functions are PascalCase.
- Keep the engine immutable: `Board` and `Game` are data classes and moves return new instances. Don't introduce mutable shared state into `engine/`.
- UI state flows down from `GameViewModel`; events flow up as lambdas. Don't reach into the view model from deep composables.
- Prefer small, focused changes. Don't refactor or reformat unrelated code in a feature PR.
- Keep existing comments and KDoc unless your change makes them wrong.
- No new dependencies without saying why in the PR description.

### Tests

- Engine changes **must** come with JVM unit tests in `app/src/test/java/com/bentrengrove/chess/engine/`. Use the helpers in `TestBoards.kt` (`sq("e4")`, `customBoard(...)`) to set up positions readably.
- Bug fixes start with a failing test that reproduces the bug.
- Don't weaken or delete an existing test to make your change pass. If a test is genuinely wrong, explain why in the PR.

### UI changes

- Add or update a `@Preview` for any composable you change.
- If your change alters what an existing screenshot test renders, update its baselines (see Screenshot tests).
- If the task is labelled `agent:device`, verify the change on the emulator and attach screenshots (and a recording for interactions or animations) to the PR's "On device" section.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bentrengrove/chess](https://github.com/bentrengrove/chess) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
