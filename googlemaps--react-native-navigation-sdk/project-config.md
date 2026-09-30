---
trigger: always_on
description: Copyright 2026 Google LLC
---

<!--
Copyright 2026 Google LLC

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

# Agent Guide for Google Navigation for React Native

This file provides instructions for AI coding agents working in this repository.
Read `README.md` and `CONTRIBUTING.md` before making substantial changes, and
follow existing code and test patterns when they are more specific than this
guide. Read `example/README.md` for sample app setup and `MIGRATING.md` for
compatibility and migration work.

## Core rules

- Keep changes focused on the requested issue. Do not perform unrelated cleanup
  or broad refactoring without a clear need.
- Preserve existing public API behavior unless the task explicitly calls for a
  breaking change.
- Maintain Android and iOS parity for cross-platform features. Document intended
  platform-specific behavior rather than silently omitting an implementation.
- This package requires React Native's new architecture: Fabric and TurboModules.
  Do not introduce a legacy-architecture fallback as an incidental fix.
- Never hand-edit generated code. Change its source definitions and regenerate
  through the appropriate build tooling.
- Add or update tests for behavior changes and bug fixes.
- Update public documentation and the example app when user-facing behavior or
  API usage changes.
- Never commit API keys, credentials, signing material, or other secrets.
- Do not claim that checks passed unless they were actually run successfully.

## Repository layout

The repository contains one React Native library and its Yarn workspace example:

- `src/`: TypeScript source for the public library.
  - `src/index.ts`: Main public export file.
  - `src/maps/`: Map types, view component, and controller.
  - `src/navigation/`: Navigation types, provider, hooks, view, and controllers.
  - `src/auto/`: Android Auto and CarPlay hook, controller access, and types.
  - `src/shared/`: Shared types, event-listener hooks, and conversion utilities.
  - `src/native/`: React Native Codegen specifications for native modules and
    the Fabric view component.
- `android/`: Android library implementation in Java and Kotlin, plus Gradle
  configuration.
- `ios/react-native-navigation-sdk/`: iOS implementation in Objective-C and
  Objective-C++, including CarPlay support.
- `react-native-navigation-sdk.podspec`: iOS package integration and native SDK
  dependency configuration.
- `lib/`: Generated CommonJS, ES module, and TypeScript declaration outputs;
  ignored by Git.
- `example/`: Sample app, native app projects, and test infrastructure.
  - `example/e2e/`: Detox test drivers and shared helpers.
  - `example/src/screens/IntegrationTestsScreen.tsx`: In-app integration tests.
  - `example/src/screens/integration_tests/`: Integration-test support code.
- `scripts/`: Native formatting and license-header scripts.
- `.github/workflows/` and `.github/actions/setup/`: CI checks and tool setup.
- `ANDROIDAUTO.md`, `CARPLAY.md`, and `MIGRATING.md`: Specialized setup and
  migration documentation.

## Development setup

Use the Node version in `.nvmrc` and the Yarn version in the root `package.json`
`packageManager` field. CI uses `.github/actions/setup/action.yml` to install
workspace dependencies:

```sh
yarn install --immutable
```

Use Yarn for repository development. Do not create or update `package-lock.json`
with npm; it is ignored here. When intentionally changing dependencies, use Yarn
and include the corresponding `yarn.lock` changes.

Additional tools depend on the files being changed:

- Android work requires the JDK version configured in `.github/workflows/ci.yml`
  and a working Android SDK. Follow the Kotlin version, new-architecture,
  Jetifier, and desugaring requirements in `README.md` and `MIGRATING.md`.
- iOS work requires macOS, Xcode, and CocoaPods. Use CI's versions when
  reproducing a CI issue rather than duplicating version pins in this guide.
- iOS uses CocoaPods for React Native integration and the podspec's
  `spm_dependency` for GoogleNavigation. Preserve this combined setup and the
  example Podfile's new-architecture and dynamic-framework configuration.
- Native formatting requires `google-java-format`, `clang-format`, and the
  Gradle-based Kotlin formatter. Consult CI and the scripts for versions.
- License checks require Google's `addlicense` tool.
- Integration tests require Detox prerequisites and an available device or
  simulator matching `example/.detoxrc.js`.

Before building or running the iOS example, install pods:

```sh
(cd example/ios && pod install)
```

Open `example/ios/SampleApp.xcworkspace`, not the project file, in Xcode.
Configure API keys following `example/README.md`:

- Android: use the ignored `example/android/local.properties` file.
- iOS: copy `example/ios/SampleApp/Keys.plist.sample` to the ignored `Keys.plist`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [googlemaps/react-native-navigation-sdk](https://github.com/googlemaps/react-native-navigation-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
