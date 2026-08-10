---
trigger: always_on
description: Guidance for AI coding agents working on this repository.
---

# AGENTS.md

Guidance for AI coding agents working on this repository.
This complements [README.md](README.md) (for humans) with agent-focused context.

## Project Overview

**Hermes Mobile** is a Jetpack Compose Android app — a mobile control panel for
[Hermes Agent](https://hermes-agent.nousresearch.com). It talks to the Hermes
dashboard's REST API and WebSocket TUI Gateway (JSON-RPC 2.0) over a trusted LAN.

- **Package:** `com.m57.hermescontrol`
- **Min SDK 26 / Target SDK 36 / Compile SDK 36**
- **Kotlin 2.4.10**, KSP 2.3.10 (standalone versioning, NOT `kotlinVersion-kspVersion`)
- **Jetpack Compose**, Room 2.7.x, Navigation3, OkHttp WebSocket, Retrofit, Kotlinx Serialization
- **Auth:** `EncryptedSharedPreferences` (AES256-GCM), Bearer token (REST) + `?token=` (WS)

## Build & Test Commands

### ✅ Local Android SDK — if available

If you have a local Android SDK (`ANDROID_HOME` set), these work:

```bash
./gradlew assembleDebug                     # full APK build
./gradlew testDebugUnitTest                 # unit tests (MockK)
./gradlew ktlintCheck                       # style check
```

**ktlint standalone** (no SDK needed):

```bash
# Download the matching binary
curl -sSLO https://github.com/pinterest/ktlint/releases/download/1.2.1/ktlint
chmod +x ktlint
./ktlint <file>                             # check one file
./ktlint --format <file>                    # auto-fix
```

**No SDK? CI handles everything** — push small and watch the checks below.

### CI pipeline (`.github/workflows/android.yml`)

| Job                  | Purpose                                                                                                                            |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| `ktlint`             | ktlint 1.2.1 style check + `checkColorLiterals` (hardcoded Color guard, issue #622)                                                |
| `android-lint`       | Android Lint                                                                                                                       |
| `unit-tests`         | JUnit                                                                                                                              |
| `build`              | assembleDebug (gated by the 3 above)                                                                                               |
| `release-compile`    | compileReleaseKotlin — catches release-variant compilation issues (e.g. debugImplementation-scoped deps referenced in main source) |
| `instrumented-tests` | Compose UI tests on Android emulator (API 34 ATD)                                                                                  |
| `ci-summary`         | Aggregator (`if: always()`) — branch protection gates on THIS check                                                                |

Also: There is a separate workflow `merge-conflict-detector.yml` that auto-labels PRs with `has-conflicts`.

Every Gradle job validates `gradle-wrapper.jar` and uses the remote build cache
(`GRADLE_ENCRYPTED_KEY` secret). `concurrency.cancel-in-progress: true`.

### Releasing (`.github/workflows/release.yml`)

Triggers on `git push tag v*`. Release APK uses R8 minification + resource shrinking.
Requires `permissions: contents: write` on the `build-release` job.

## Code Style

- **ktlint 1.2.1** is enforced in CI. Run `./ktlint --format` before pushing.
- Import ordering is the #1 CI failure: ktlint enforces ASCII-lexicographic order
  (uppercase before lowercase: `LaunchedEffect` before `collectAsState`).
- `const val` must use SCREAMING_SNAKE_CASE.
- Trailing commas required. No trailing whitespace.
- 120 char max line length.

## Architecture Conventions

### Navigation (Navigation3 — NOT Navigation Compose)

Uses `androidx.navigation3` (`NavKey`, `NavBackStack`, `NavDisplay`, `entry<T>`).

**⚠ Always route navigation through `NavigationController.navigateTo()`.** Never call
`backStack.add(ScreenKey)` directly from UI callbacks — the controller has a
deduplication guard. Bypassing it stacks duplicate screen entries that compete for
touch events and become unresponsive.

**Drawer gesture state is screen-owned (issue #619).** Each screen declares whether
the modal drawer's swipe gestures should be active while it is visible, via a single
source of truth — no global gesture set, no defensive `LaunchedEffect(snapTo(Closed))`,
no `closeDrawer` callback on `NavigationController`.

- `HermesScaffold(drawerGesturesEnabled = true)` — default; primary screens and
  most list screens. The scaffold reconciles this preference into the
  `DrawerGestureController` via a `SideEffect`.
- `HermesScaffold(drawerGesturesEnabled = false)` — drill-down sub-pages (e.g.
  `SettingsConnectionPage`, `SettingsAppearancePage`, …). The controller closes
  the drawer itself if it was open when the screen composed, so the scrim can't
  stick around and intercept the next tap.
- `DisableDrawerGestures()` — for entry screens that don't use `HermesScaffold`
  (Landing, AuthLogin, PairingCodeEntry).

`ModalNavigationDrawer` in `Navigation.kt` reads `gestureController.enabled`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hy4ri/hermes-mobile](https://github.com/Hy4ri/hermes-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
