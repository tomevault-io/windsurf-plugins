---
trigger: always_on
description: Quotio is a native macOS menu bar and window app for operating CLIProxyAPI. It manages
---

# AGENTS.md

## Project

Quotio is a native macOS menu bar and window app for operating CLIProxyAPI. It manages
the local proxy lifecycle, provider OAuth accounts, quota monitoring, CLI agent
configuration, tunnels, and updates.

- Swift 6 and SwiftUI, with targeted AppKit integration
- Minimum deployment target: macOS 14.0
- Xcode project: `apps/macos/Quotio.xcodeproj`; shared scheme: `Quotio`
- Targets: `Quotio` (application) and `QuotioTests` (executable integration tests)
- Core package: `Packages/QuotioCore` with four source and four test targets
- Dependency manager: Swift Package Manager through the local package and Xcode project
- Package: Sparkle 2.8.1
- No CocoaPods, Carthage, Fastlane, root Swift package, or UI-test target

The app uses Clean Architecture module boundaries with pragmatic MVVM in Presentation.
`QuotioDomain` owns values and rules, `QuotioApplication` owns use cases and ports,
`QuotioInfrastructure` owns side effects and SDK adapters, and `QuotioPresentation`
owns SwiftUI/AppKit views and observable screen state. The executable composes the
production graph and owns lifecycle only.

## Project Map

- `Packages/QuotioCore/Sources/QuotioDomain/`: entities, value types, settings values,
  and pure policies.
- `Packages/QuotioCore/Sources/QuotioApplication/`: use cases, feature controllers,
  and side-effect ports.
- `Packages/QuotioCore/Sources/QuotioInfrastructure/`: HTTP, filesystem, process,
  Keychain, SQLite, Sparkle, OAuth, provider, proxy, agent, and tunnel adapters.
- `Packages/QuotioCore/Sources/QuotioPresentation/`: SwiftUI/AppKit views, observable
  screen models, settings managers, menu bar UI, and localization helpers.
- `Packages/QuotioCore/Tests/`: unit and regression tests, split by owning module.
- `apps/macos/Quotio/QuotioApp.swift`: SwiftUI scene entry point.
- `apps/macos/Quotio/App/`: `CompositionRoot`, `AppRuntime`, `AppDelegate`, and app-only adapters.
- `apps/macos/Quotio/Assets.xcassets`: app icon, accent color, provider art, and menu bar assets.
- `apps/macos/Quotio/Localizable.xcstrings`: String Catalog for `en`, `fr`, `vi`, and `zh-Hans`.
- `apps/macos/Quotio/Info.plist`, `apps/macos/Quotio/Quotio.entitlements`: app metadata and entitlements.
- `apps/macos/QuotioTests/`: executable dependency-graph, lifecycle, identity, and bundle tests.
- `apps/macos/Config/`: Debug/Release xcconfig files and the template for local overrides.
- `apps/macos/scripts/`: local build/run helpers and release packaging scripts.
- `.github/workflows/macos-release.yml`: tag/manual release pipeline.

The Xcode groups and local package use filesystem synchronization. Place new files in
the owning module or test target; manual edits to `project.pbxproj` are normally
unnecessary.

## Build, Test, and Run

Run commands from the repository root. Choose the check that exercises the changed
behavior; the command list is a reference, not a checklist for every task. Report
results from the current checkout rather than assuming a previous run still applies.

Build the Debug app:

```bash
xcodebuild -project apps/macos/Quotio.xcodeproj -scheme Quotio -configuration Debug -destination 'platform=macOS' build
```

For package-wide changes, run all package tests. Run architecture checks when module
boundaries, imports, or dependency composition change:

```bash
swift test --package-path Packages/QuotioCore
./apps/macos/scripts/check_architecture.sh
```

For executable-wide changes, run the complete integration-test target:

```bash
xcodebuild -project apps/macos/Quotio.xcodeproj -scheme Quotio -configuration Debug -destination 'platform=macOS' test
```

Run one executable integration test class (replace the class name as needed):

```bash
xcodebuild -project apps/macos/Quotio.xcodeproj -scheme Quotio -configuration Debug -destination 'platform=macOS' -only-testing:QuotioTests/AppRuntimeTests test
```

Build, terminate any running Quotio instance, launch the new Debug app, and verify that
the process remains running:

```bash
./apps/macos/scripts/build_and_run.sh --verify
```

The run script leaves Quotio running and writes derived data under
`apps/macos/build/DebugDerivedData`. Always pass `--package-path Packages/QuotioCore` to SwiftPM;
the repository root is not a Swift package.

## Architecture and Coding Conventions

- Follow the existing naming scheme: `*Screen`, `*ViewModel`, `*Service`, `*Manager`,
  and provider-specific `*QuotaFetcher`. Use UpperCamelCase for types and lowerCamelCase
  for members.
- Keep one primary type per file and place it in the directory that owns its role.
  Extract a component or service only when it has a coherent responsibility.
- Keep Domain independent, Application dependent only on Domain, Infrastructure
  dependent on Application and Domain, and Presentation dependent on Application and
  Domain. Only `CompositionRoot` may construct concrete Infrastructure adapters.
- Do not add first-party singleton services. Inject long-lived dependencies from the
  composition root and pass platform behavior through Application ports.
- Use the existing Observation flow: `@Observable` models/view models, `@Environment`
  for shared dependencies, `@State` for view-owned state, and `@Bindable` when bindings

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nguyenphutrong/quotio](https://github.com/nguyenphutrong/quotio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
