---
trigger: always_on
description: Guidance for AI agents (and humans) working in this repository. This is the
---

# CLAUDE.md

Guidance for AI agents (and humans) working in this repository. This is the
canonical onboarding doc — `AGENTS.md`, `README.md`, and `CONTRIBUTING.md` point here.

## What this is

`react-native-restart` is a React Native native module that reloads the running JS
bundle at runtime (a programmatic "restart the app"). It is published to npm as
[`react-native-restart`](https://www.npmjs.com/package/react-native-restart).

It is a **thin JS bridge over three native implementations** (iOS, Android, Windows).
There is almost no JS logic — the JS layer just forwards calls to a native module named
`RNRestart` that exists on each platform.

Public API (see `src/index.tsx`):

| Method | Notes |
| --- | --- |
| `RNRestart.restart(reason?)` | Restart the app. Preferred entry point. Android does a full process restart; iOS/Windows reload the bundle. |
| `RNRestart.Restart(reason?)` | **Deprecated** — kept for backward compatibility; identical behavior. |
| `RNRestart.getReason()` | `Promise<string \| null>` — returns the last reason passed to a restart (survives the Android process restart). |

## Architecture

Four layers that must stay in sync:

```
JS bridge        src/index.tsx           ← NativeModules.RNRestart, default export RNRestart
   │             src/NativeRNRestart.ts   ← TurboModule spec (drives New-Architecture codegen)
   │
   ├─ iOS        ios/Restart.m|.h         ← RCTTriggerReloadCommandListeners (main thread)
   ├─ Android    android/src/main/java/com/reactnativerestart/
   │                 RestartModule.java       (ReactContextBaseJavaModule, name "RNRestart")
   │                 RestartPackage.java       (ReactPackage registration)
   │                 ReactInstanceHolder.java  (instance-manager holder interface)
   └─ Windows    windows/ReactNativeRestart/  (C++/WinRT, New-Architecture TurboModule + codegen)
```

- **iOS**: `Restart`/`restart` store the reason, hop to the main thread, and call
  `RCTTriggerReloadCommandListeners(...)` to reload the bundle.
- **Android**: `RestartModule` does a **full process restart** via
  `ProcessPhoenix.triggerRebirth(...)` (the `com.jakewharton:process-phoenix` dependency in
  `android/build.gradle`). The reason is persisted to `SharedPreferences` (synchronous
  `commit()`, since the process is killed immediately) and read back on next launch, so
  `getReason()` still works across the restart.
- **Windows**: New-Architecture TurboModule; the spec in `src/NativeRNRestart.ts` is
  codegen'd into `windows/ReactNativeRestart/codegen/` (see `codegenConfig` in `package.json`).

> **Key rule:** changing the public API means editing the JS type in `src/index.tsx`,
> the TurboModule spec in `src/NativeRNRestart.ts`, **and every native platform in
> lockstep** (iOS, Android, Windows). The native module name is `RNRestart` everywhere —
> don't rename it on one platform only.
>
> Note the platforms differ by design: Android performs a full **process** restart, while
> iOS/Windows reload the **bundle**.

## Repository map

| Path | What it is |
| --- | --- |
| `src/index.tsx` | The JS/TS surface. `RestartType` + default export. |
| `src/NativeRNRestart.ts` | TurboModule spec consumed by RN codegen (New Architecture). |
| `src/__tests__/index.test.tsx` | Jest unit test for the JS API. |
| `src/__mocks__/react-native-restart.tsx` | Jest manual mock consumers use in their tests. |
| `ios/` | iOS native module (`Restart.m`, `Restart.h`) + Xcode project. |
| `android/` | Android library module (`build.gradle`, Java sources under `src/main/java/...`). |
| `windows/` | Windows (RNW) New-Architecture module, incl. generated `codegen/`. |
| `Example/` | Standalone RN app that consumes the library; has its own package.json/tests. |
| `react-native-restart.podspec` | CocoaPods spec (reads version from `package.json`). |
| `badges/` | Auto-generated coverage badges (`yarn test` output — do not hand-edit). |
| `lib/` | **Build output** from `react-native-builder-bob` — do not hand-edit. |
| `.github/workflows/` | CI (`ci.yml`), PR checks (`pull_request.yml`), release (`publish.yml`). |

## Common commands

Run from the repo root (yarn is the project's package manager; `npm run <script>` also works):

| Command | What it does |
| --- | --- |
| `yarn test` | Jest with coverage, then regenerates coverage badges. |
| `yarn typescript` | Type-check with `tsc --noEmit`. |
| `yarn lint` | ESLint over `.js/.ts/.tsx`. |
| `yarn bootstrap` | Install deps for root + `Example/`, run pods, copy `.env`. |
| `yarn example <cmd>` | Run a script inside the `Example/` app (e.g. `yarn example start`). |
| `bob build` (`yarn prepare` / `prepack`) | Compile `src/` → `lib/` (commonjs, module, typescript). |

Run `yarn typescript && yarn lint && yarn test` before committing — the same checks CI runs.
(`yarn` and `npm run` are interchangeable here; the checks above only need Node + `npm ci`.)

## Local environment setup (macOS)

The JS checks only need Node. To run the **full native build matrix** (what CI's
`build-android` / `build-ios` jobs do) you also need Java, the Android SDK, Xcode,
and CocoaPods. One-time setup with Homebrew:

```bash
# JS/native tooling
brew install openjdk@17 cocoapods watchman

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [avishayil/react-native-restart](https://github.com/avishayil/react-native-restart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
