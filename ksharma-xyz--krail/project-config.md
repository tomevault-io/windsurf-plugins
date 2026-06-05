---
trigger: always_on
description: Compose Multiplatform app targeting Android + iOS.
---

# KRAIL — Claude Project Notes

## Project

Compose Multiplatform app targeting Android + iOS.
Android is the primary testable target from the command line. iOS tests are not run.

## Test Commands

| Scope | Command |
|---|---|
| Single module | `./gradlew :feature:track:ui:testAndroidHostTest` |
| Multiple modules | `./gradlew :a:testAndroidHostTest :b:testAndroidHostTest --continue` |
| All modules | `./gradlew testAndroidHostTest --continue` |

**Wrong tasks (do not use):** `jvmTest`, `testDebugUnitTest`, `allTests`

Modules use KMP `androidLibrary { withHostTest {} }` — this creates `testAndroidHostTest`, not the standard AGP task name.

Modules that have `withHostTest {}` enabled:
- `feature/trip-planner/ui`
- `feature/track/ui`
- `feature/track/state`

If a module is missing `testAndroidHostTest`, add `withHostTest {}` inside its `androidLibrary {}` block in `build.gradle.kts`.

## Detekt

```
./gradlew detekt --continue
```

`autoCorrect: true` is set in `detekt.yml` — import ordering and trailing commas are fixed in-place automatically.

Suppression rules:
- Break long lines instead of suppressing `MaximumLineLength` / `MaxLineLength`
- Extract constants instead of suppressing `MagicNumber` (unless truly no reuse value)
- Only suppress `CyclomaticComplexMethod` / `LongMethod` when refactoring is genuinely not possible

## LazyColumn / LazyRow item keys

**Always provide an explicit `key` for every `item {}` call** — this is critical for correct
recomposition, scroll-state preservation, and animation behaviour.

```kotlin
// ✅ correct — stable, unique key per item
item(key = "origin-destination") { ... }
item(key = "spacer-top") { ... }
items(journeys, key = { it.journeyId }) { ... }

// ❌ wrong — no key means Compose uses positional identity, which breaks on reorder/insert
item { ... }
```

Key rules:
- Static items use a descriptive string literal (`"spacer-top"`, `"load-more-button"`)
- Dynamic items use a stable domain identifier (e.g. `journeyId`, `stopId`)
- When the same data appears twice in the same list (e.g. previous journeys + main journeys),
  prefix keys to keep them unique: `"prev_$journeyId"` vs plain `journeyId`

## Pull Requests

**Always use Graphite (`gt submit`) to raise PRs — never `gh pr create` directly.**

We stack PRs. Break work into focused, layered branches and submit the full stack with `gt submit --stack --publish`.

**Max 500 lines of change per PR.** If a branch exceeds this, split it before submitting:
- Use `gt branch split --by-commit` or carve out a new child branch
- Each PR should have a single clear concern (ViewModel logic, UI layer, bug fix, etc.)

**Before raising a PR (or when asked to "fix issues and push" / "run quality checks"), run locally and fix all failures first:**

1. Detekt — catches style, formatting, and lint issues before CI does:
   ```
   ./gradlew detekt --continue
   ```
   After detekt runs, always check for auto-corrected files and commit them:
   ```
   git diff --name-only   # stage + commit any files detekt auto-corrected
   ```
   `autoCorrect: true` silently rewrites source files on disk. If those changes
   aren't committed, CI sees the original violations and fails even though local
   detekt reported success.

2. Unit tests — run tests for every module touched by the change:
   ```
   ./gradlew :module:a:testAndroidHostTest :module:b:testAndroidHostTest --continue
   ```

Both must be green before submitting the PR.

## Build

Never run build/compile commands (assembleDebug, etc.) — ask the user to run them and share output.

## Submodules

KRAIL pulls in the `krail-api-proto` repo as a git submodule at `krail-api-proto/`.
Wire codegen in `:io:bff-api` reads `.proto` files from there. If a fresh checkout
or worktree shows the directory empty, run:

```sh
git submodule update --init --recursive
```

`compileDebugSources` fails with a "no protos found" error otherwise. CI workflows
that compile pass `submodules: true` to `actions/checkout`; if you add a new
workflow that compiles, do the same.

## Worktree build setup

Fresh worktrees are missing gitignored files and build artefacts required to compile
`:androidApp`. Before asking the user to run any build in a worktree, copy all four
of these from the main checkout (`/Users/ksharma/code/apps/KRAIL/`):

```sh
WORKTREE=/Users/ksharma/code/apps/KRAIL/.claude/worktrees/<name>
MAIN=/Users/ksharma/code/apps/KRAIL

# 1. Gradle local config
cp $MAIN/local.properties $WORKTREE/local.properties

# 2. Firebase config (three locations)
cp $MAIN/androidApp/src/debug/google-services.json   $WORKTREE/androidApp/src/debug/google-services.json
cp $MAIN/androidApp/src/release/google-services.json $WORKTREE/androidApp/src/release/google-services.json
cp $MAIN/androidApp/src/main/google-services.json    $WORKTREE/androidApp/src/main/google-services.json
cp $MAIN/composeApp/src/debug/google-services.json   $WORKTREE/composeApp/src/debug/google-services.json
cp $MAIN/composeApp/src/release/google-services.json $WORKTREE/composeApp/src/release/google-services.json

# 3. Wire-generated proto sources (saves a full codegen run)
cp -R $MAIN/io/bff-api/build/generated $WORKTREE/io/bff-api/build/generated

# 4. Proto submodule
git -C $WORKTREE submodule update --init --recursive
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ksharma-xyz/KRAIL](https://github.com/ksharma-xyz/KRAIL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
