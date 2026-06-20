---
trigger: always_on
description: >
---


# App Tester

Tests app navigation flows by driving the device with `agent-device`, a unified CLI for iOS,
Android, macOS, tvOS, and Android TV (real devices and simulators).

**Strategy**

1. Read the project's navigation source to build `.tester/app-graph.yaml` and
   `.tester/flows/*.yaml` at the project root.
2. Instrument screen files with structured print/console.log lines and accessibility
   identifiers (`testID` / `Semantics` / `.accessibilityIdentifier()`).
3. Drive flows by `open` → `snapshot -i` → `press`/`fill`/`scroll` → confirm via console
   logs → `close` — all via `agent-device`.
4. Take screenshots **only when a step fails**, and write them to `/tmp/` —
   **never** under `.tester/`. The `.tester/` directory is reserved for the graph
   (`app-graph.yaml`) and flow YAML files; screenshots, snapshots, and logs are
   ephemeral debugging artifacts and belong in `/tmp/`.

## Requirements

- **`agent-device`** — the only runtime dependency. `npm install -g agent-device` (Node ≥22).
- Platform SDKs for the **build** step only:
  - iOS / macOS / tvOS: Xcode + Command Line Tools.
  - Android: Android SDK + ADB on PATH (`$HOME/Library/Android/sdk/platform-tools`).
  - Flutter: `~/fvm/versions/stable/bin/flutter` or `flutter` on PATH.
  - Expo / React Native: Node + the project's package manager.
- macOS Accessibility permission for desktop automation (System Settings → Privacy &
  Security → Accessibility, on first run).

## Routing into agent-device help

`agent-device` is the source of truth for command syntax — it ships versioned help that this
skill is intentionally not duplicating. Whenever you need exact flags or new commands, run:

```bash
agent-device --version
agent-device help workflow      # canonical session loop + selectors
agent-device help debugging     # logs, network, performance, react-devtools
agent-device help macos         # macOS-specific notes
agent-device help remote        # device farms / cloud
```

If `agent-device --version` reports `< 0.14.0`, upgrade before using this skill — older CLIs
lack the help topics above.

## Canonical session loop

```text
open <app>  →  snapshot -i  →  get / is / find  →  press / fill / scroll / wait  →  verify  →  close
```

Snapshots assign refs (`@e1`, `@e2`, …) to current-screen elements. Refs from the most recent
snapshot are immediately actionable. If a target appears only in an off-screen summary, use
`scroll <direction>` and re-snapshot until it's visible.

A minimal flow step:

```bash
agent-device open com.example.YourApp --platform ios
agent-device snapshot -i                                 # find @e for the action
agent-device press --label "Create Game"                 # or: press @e3
# verify via app logs (see "Log confirmation" below)
agent-device close
```

> Always look up exact selectors and flags via `agent-device help workflow` rather than guessing.

---

## Sensitive Credentials (.env)

Some flows require authentication. Store credentials in `.env` at the project root (add to
`.gitignore`):

```bash
TEST_USERNAME=your@email.com
TEST_PASSWORD=yourpassword
TEST_PERMISSIONS=camera,location,notifications      # iOS only
SYSTEM_PROMPT_DISMISS=Ask App Not to Track,Don't Allow,Allow Once,Not Now,Dismiss,OK,Allow
```

Load before testing: `export $(grep -v '^#' .env | xargs)`

---

## Step 0: Project Setup

Identify these values once before any phase.

**iOS / macOS / tvOS (SwiftUI / UIKit):**

| Value | How to find it |
|---|---|
| **Platform** | `SUPPORTED_PLATFORMS` in build settings — `macosx`, `iphonesimulator`, `appletvsimulator` |
| **Bundle ID** | `PRODUCT_BUNDLE_IDENTIFIER` in build settings or Info.plist |
| **App name** | `CFBundleDisplayName` / `CFBundleName` in Info.plist or Xcode scheme name |
| **Screen files** | Directory containing `*View.swift` or `*ViewController.swift` |
| **Navigation source** | File with Screen/Route enum or coordinator |
| **Log prefix** | `[AppName]` — used in all instrumentation |

**Android / Flutter:**

| Value | How to find it |
|---|---|
| **Platform** | Presence of `android/`. Flutter = `pubspec.yaml` present |
| **Package ID** | `applicationId` in `android/app/build.gradle(.kts)` |
| **App name** | `android:label` in `AndroidManifest.xml` |
| **Screen files** | Flutter: `lib/features/*/screens/` or `lib/screens/` (`*Screen.dart` / `*Page.dart`) |
| **Navigation source** | Flutter: GoRouter config, or files with `GoRoute` / `Navigator.push` calls |
| **Device serial** | `adb devices` — e.g. `emulator-5554` |
| **Log prefix** | `[AppName]` — used in `print()` calls |

**Expo / React Native (iOS + Android):**

Identify by `package.json` containing `expo` or `react-native`. Expo Router projects also have
an `app/` directory with file-based routes.

| Value | How to find it |
|---|---|
| **Platform** | `package.json` has `expo` → Expo. `react-native` only → bare RN |
| **Bundle ID (iOS)** | `app.json` → `expo.ios.bundleIdentifier`, or `ios/<App>/Info.plist` |
| **Package ID (Android)** | `app.json` → `expo.android.package`, or Android `applicationId` |
| **App name** | `app.json` → `expo.name`, or `app.config.{js,ts}` |
| **Router type** | `app/` directory exists → **Expo Router** (file-based). Otherwise `@react-navigation/*` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [markgravity/app-tester-skill](https://github.com/markgravity/app-tester-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
