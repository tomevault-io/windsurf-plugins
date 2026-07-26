---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ZODL (formerly Zashi) is an iOS Zcash wallet built with SwiftUI and The Composable Architecture (TCA). It uses the Zcash Swift SDK (`ZcashLightClientKit`) for blockchain operations.

## App name

The app's name is always written **ZODL** — all uppercase. Whenever generated text refers to the app by name — UI strings (`Localizable.xcstrings`), code, comments, documentation, commit messages, PR titles/descriptions, etc. — it MUST be `ZODL`, never `Zodl` (nor `zodl`/`ZODl`). This rule is about the app name as a word; it does NOT change fixed technical identifiers such as the `zodl_internal` module, the `zodl-ios` repository, scheme names, or bundle IDs. The former name "Zashi" is unaffected.

## Build & Development

**Prerequisites:** Install SwiftGen (`brew install swiftgen`) and SwiftLint (v0.50.3 specifically - use the official .pkg installer). Both run automatically during Xcode builds.

**Build targets / schemes:**
- `zodl-internal` - internal/development build (mainnet/ZEC); its scheme also runs the `zodlTests` test target
- `zodl-testnet` - testnet build (TAZ token)
- `zodl-production` - production / App Store build (mainnet/ZEC), built via the `zodl-AppStore` scheme
- `zodlTests` - test target (run via the `zodl-internal` scheme)
- Conditional compilation via `SECANT_MAINNET` / `SECANT_TESTNET` (plus `SECANT_DISTRIB` on the App Store config). These flags keep the legacy `SECANT_` prefix even though the targets are now named `zodl-*`.

**Build:** Open `secant.xcodeproj` in Xcode and build the desired scheme (CI builds with `xcodebuild -project secant.xcodeproj`).

**Tests:** Run the `zodlTests` target in Xcode (scheme `zodl-internal`). All tests use **Swift Testing** (`@Suite` / `@Test` / `#expect` / `#require`) — **write every new test in Swift Testing, never XCTest.** Tests use TCA's `TestStore` with dependency injection (`.noOp`, `withDependencies`, etc.). Swift Testing runs suites in parallel by default, so mark any suite that mutates process-global state (named `UserDefaults` suites, the OSLog store, shared singletons / TCA `@Shared` state) with `@Suite(.serialized)`.

**Linting:** SwiftLint runs as a build phase. Config: `.swiftlint.yml` (app code) and `.swiftlint_tests.yml` (tests, more relaxed). Key enforced rules: no string concatenation (use interpolation), no `NSLog`, no `print`/`debugPrint` in app code, TODOs must reference issue numbers (`TODO: [#123]`).

## Architecture

**TCA (The Composable Architecture)** drives all state management, using modern macros (`@Reducer`, `@ObservableState`, `@Dependency`). Each feature has:
- `<Feature>Store.swift` - State, Action, Reducer, dependencies
- `<Feature>View.swift` - SwiftUI view consuming the store
- `<Feature>Coordinator.swift` (some features) - Navigation glue between screens

**Source layout** (`secant/Sources/`):
- `Features/` - Screen-level features (~40), each in its own directory
- `Features/CoordFlows/` - Multi-screen coordinator flows (Send, Restore, Scan, SwapAndPay, AddKeystoneHWWallet, RequestZec, SignWithKeystone, Transactions, Voting, WalletBackup). Each flow has `<Name>CoordFlowStore.swift`, `<Name>CoordFlowView.swift`, and `<Name>CoordFlowCoordinator.swift`. Most are flat files directly in `CoordFlows/`; Voting lives in its own `VotingCoordFlow/` subdirectory.
- `Dependencies/` - Dependency clients (~49) wrapping SDK, iOS, and custom services
- `UIComponents/` - Reusable UI building blocks (buttons, text fields, badges, etc.)
- `Models/` - Shared data types (TransactionState, StoredWallet, WalletAccount, SwapAsset, Swaps, WalletStatus, etc.)
- `Utils/` - Helpers and extensions
- `Generated/` - SwiftGen output (assets, fonts) - do not edit manually
- `Resources/` - Assets, fonts (Inter, RobotoMono, Zboto, Michroma), Lottie animations, localizations

**Root feature** (`Features/Root/`) is the app coordinator - handles wallet initialization, navigation, and deep linking across 12 files.

**Dependencies** use the `@DependencyClient` macro from `swift-dependencies` on a struct with `@Sendable` closures (Swift 6 concurrency). Layout per client:
- `<Name>Interface.swift` - `@DependencyClient struct <Name>Client { ... }` plus the `DependencyValues` extension
- `<Name>LiveKey.swift` - `liveValue` conformance for production
- `<Name>TestKey.swift` - **only when** the macro-generated default isn't enough; otherwise omit (the macro provides `testValue` automatically). Tests can also override individual closures inline via `withDependencies`.

Closures must be `@Sendable`. Use `@preconcurrency import ZcashLightClientKit` when an SDK type is not yet `Sendable`.

**Transaction guard (`Dependencies/TransactionGuard/`)** — the SDK's `switchTo(endpoint:)` tears down and rebuilds the synchronizer, so it must never overlap a transaction broadcast. A shared, non-reentrant FIFO-mutex actor (`@Dependency(\.transactionGuard)`) enforces this **inside the dependency LiveKeys** — feature code never wraps broadcasts; the one feature-level guard use is the manual switch (`switchWaiting`) in `ServerSetupStore`:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zodl-inc/zodl-ios](https://github.com/zodl-inc/zodl-ios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
