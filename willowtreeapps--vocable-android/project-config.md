---
trigger: always_on
description: AAC (Augmentative and Alternative Communication) app for users with motor/speech impairments. Primary input is ARCore head/face tracking rendered as a gaze cursor with dwell-click, with touch as fallback. Fully offline/local-first — no backend, no network layer.
---

# Vocable Android

AAC (Augmentative and Alternative Communication) app for users with motor/speech impairments. Primary input is ARCore head/face tracking rendered as a gaze cursor with dwell-click, with touch as fallback. Fully offline/local-first — no backend, no network layer.

See `Documentation/architecture-diagrams.md` for the user-journey and tech-stack diagrams that accompany this baseline.

## Cross-repo: vocable-ios

This repo has a sibling iOS repo, `willowtreeapps/vocable-ios` (default branch `develop` — not `main`). **iOS development is ahead of Android** — new features typically land on iOS first. When a ticket asks Android to add or match a feature that already exists on iOS, check the Swift implementation for the intended UX/behavior/edge cases before designing the Android version, rather than reinventing it from scratch.

**The shipped iOS implementation is the source of truth for design and behavior — not Figma.** Where a Figma mock and the iOS code disagree, follow iOS and flag the divergence back to design on the relevant design ticket rather than building the mock. Figma is a useful reference for anything iOS doesn't cover, but it lags and has been wrong: on #636 the mock called for a 3px selected-state border where iOS uses a trailing checkmark, showed one column on tablet landscape where iOS uses two, and showed per-voice proper names (`"Aria (Enhanced)"`) that Android TTS cannot produce at all. Also confirm parity is even *achievable* before committing to it — platform APIs differ, and where they do, say so on the ticket instead of forcing a match.

**Reading the iOS source.** Clone locations vary by machine — it's often cloned alongside this repo at `../vocable-ios`, but don't assume that. Check for a local clone first and use it if present. If there isn't one, the repo is public, so read it without cloning rather than asking someone to clone it:

```
gh api "repos/willowtreeapps/vocable-ios/git/trees/develop?recursive=1" --jq '.tree[].path' | grep -i <feature>
gh api "repos/willowtreeapps/vocable-ios/contents/<path>?ref=develop" --jq '.content' | base64 -d
```

Either way, start with the iOS repo's own `CLAUDE.md` for orientation, and note that localized copy lives in `Vocable/Supporting Files/Localizable.xcstrings` — it's JSON, so parse it rather than grepping.

Confirmed architecture differences (don't port 1:1): iOS is UIKit+Combine+singletons+Core Data with no ViewModel layer and no DI framework — nothing like Koin/Compose/MVI. iOS solves "editing a preset phrase" by mutating one row in place (`isUserRenamed` flag) instead of Android's shadow-phrase duplicate-row approach, so iOS has no analog to Android's PR #611 shadow-phrase sort-order bug class. iOS's dwell-click is *not* gated on TTS completion the way Android's `GazeClickable` is — treat that as a platform-behavior question to confirm with product, not an automatic bug on either side.

## Modules

- `:app` — the entire application (single module: UI, data, domain, DI). `com.willowtree.vocable` package.
- `:basetest` — thin shared test-fixture module (depends on `:app`, which is unusual but intentional). Currently only 2 files; most test fakes actually live inline under `app/src/test`.
- `build-logic` (included build) — Gradle convention plugins (`vocable.application`, `vocable.library`) centralizing compileSdk 36 / minSdk 24 / Java+Kotlin 17. Only toolchain config is centralized here — Compose/KSP/Koin/signing are configured per-module in `app/build.gradle.kts`.

No product flavors. Just `debug` (has `USE_HEAD_TRACKING` BuildConfig flag, toggle with `-PUSE_HEAD_TRACKING`) and `release`.

## Architecture

- **DI**: Koin (not Hilt/Dagger). One flat module: `di/AppKoinModule.kt`. ViewModels injected via `by viewModel()` (Activities) / `koinViewModel()` (Compose). `MainActivity`/`SplashActivity` have Koin activity-scopes for things like `FaceTrackingManager`/`PermissionsChecker`.
- **UI**: 100% Jetpack Compose, no XML layouts, no Fragments. Single `NavHost` in `ui/VocableNavHost.kt` using raw string routes with manual `URLEncoder`/`URLDecoder` for args (no typed nav-args).
- **Presentation pattern**: MVI via `ui/base/BaseViewModel.kt` (`StateFlow` for state + buffered `Channel` for one-shot events) and `ui/base/MviScreen.kt`. **Not all ViewModels follow this** — `PresetsViewModel`, `SplashViewModel`, `SensitivityViewModel` use ad hoc `StateFlow`+`LiveData` instead. When touching those, prefer migrating to `BaseViewModel` over adding more ad hoc state, but don't do a drive-by rewrite unrelated to the ticket.
- **Package layout**: `core/` (cross-cutting: prefs, TTS, face tracking, permissions, locale), `data/room/` + `data/repository/` (Room DAOs/DTOs + repos), `domain/model/` + `domain/usecase/` (interface+impl pairs, e.g. `ICategoriesUseCase`/`CategoriesUseCase`), `di/`, `ui/<feature>/` (one folder per screen).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [willowtreeapps/vocable-android](https://github.com/willowtreeapps/vocable-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
