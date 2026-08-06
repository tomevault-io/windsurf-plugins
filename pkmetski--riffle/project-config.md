---
trigger: always_on
description: Do not `assembleDebug` (or any APK build) unless the user explicitly asks. Do not `adb install` a build onto a device or emulator unless the user explicitly asks. Editing code and running JVM tests are fine without building or installing.
---

# Riffle — Agent Instructions

## Building and installing APKs

Do not `assembleDebug` (or any APK build) unless the user explicitly asks. Do not `adb install` a build onto a device or emulator unless the user explicitly asks. Editing code and running JVM tests are fine without building or installing.

## Running harness tests

Always run harness tests via `make harness-test` (phone-form-factor tests) or `make harness-test-tablet` (tests annotated with `@TabletLayout`). Never call `./gradlew :app:connectedDebugAndroidTest` directly — it targets all connected devices and will interfere with the developer's physical device. Each target boots its dedicated AVD ("Harness Medium Phone" or "Harness Medium Tablet"), runs its filtered test subset against it exclusively, then shuts it down. The two subsets are mutually exclusive, so tests never double-run across targets.

## Debugging on the developer's device

When debugging with `[DEBUG-<tag>]` logs the user is reproducing for you, **fetch the logcat yourself** — don't ask the user to paste it. The user's device is connected via `adb`; run e.g. `adb logcat -d | grep DEBUG-<tag>` (use `-d` to dump and exit, not stream). Clear the buffer with `adb logcat -c` before they reproduce so the output isn't drowned in history. If multiple devices/emulators are connected, pick the right one with `-s <serial>` (`adb devices`). Trust the user when they say "reproduced" — go fetch.

Assume the user is testing on the correct/latest version of the app. Don't diagnose a reported bug as "wrong APK installed" based on the emulator's build SHA, the drawer version string, or a missing symbol in the installed commit — the user is typically reproducing on their physical device (or a fresh install), not on the emulator you're driving. Treat the bug as real and dig into the code.

## Reader mode changes

The reader has three modes: **paginated**, **vertical**, and **continuous**.

Paginated and vertical both use Readium's `EpubNavigatorFragment` (scroll=false vs scroll=true). Readium drives navigation, emits position updates, and populates Locator fields automatically.

Continuous uses a custom `ContinuousReaderView` with a fully manual position pipeline. Anything Readium provides for free to paginated/vertical must be explicitly computed and threaded through the continuous `onPositionChanged` lambda in `EpubReaderScreen.kt`.

Any change that affects reading behaviour — typography, scrolling, navigation, decorations, layout, text size, margins, position tracking, navigation events, new ViewModel state, or UI driven by the current locator — must be verified to work in all three modes. Continuous has distinct scroll mechanics (native Android scroll vs. Readium column pagination) and separate JS injection paths; a fix that works in paged mode often breaks or is a no-op in continuous mode and vice versa. If paginated/vertical get something from Readium, ask whether continuous needs to compute an equivalent.

## Database migrations

When adding a new Room migration:

1. Bump `version` in the `@Database` annotation in `RiffleDatabase.kt` and write the new `MIGRATION_N_(N+1)` companion object.
2. Build the project so KSP exports the new schema JSON to `core/database/schemas/com.riffle.core.database.RiffleDatabase/<N+1>.json`.
3. Register the new migration in `DataModule.kt` inside `addMigrations(...)`.
4. Open `core/database/src/androidTest/kotlin/com/riffle/core/database/MigrationTest.kt` and add:
   - A new `@Test fun migrationNToN1()` following the pattern of existing tests:
     - `helper.createDatabase(TEST_DB, N)` and insert rows exercising every column touched by the migration
     - `helper.runMigrationsAndValidate(TEST_DB, N+1, true, RiffleDatabase.MIGRATION_N_(N+1))`
     - Cursor assertions verifying new columns have correct default values and all pre-existing data is preserved
   - Add the new migration to the `migrateFullChain` test's `runMigrationsAndValidate` call.

## Commit every uncommitted change on the branch

When finalizing, `git status` is almost never empty — the user routinely piggy-backs work in progress onto whatever branch is in flight. Every modified or untracked file that isn't your own scratch/debug artifact belongs on the branch and must be committed before the rebase / PR. **Unrelated ≠ unwanted.** Never stash-and-pop across the rebase (it silently drops the WIP from the PR), and never stop to ask "is this related?" — the answer is that the user knew the state of their tree when they invoked `/finalize`.

- If a change is clearly part of the same fix, fold it into the pending commit.
- If it's a whole separate feature or unrelated fix, give it its own commit with a descriptive message and mention it in the PR body so it's not a surprise in the diff.
- Only stop and ask when a file looks genuinely suspicious (potential secret, large binary that doesn't belong).

## Reference the source issue in the PR


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pkmetski/riffle](https://github.com/pkmetski/riffle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
