---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Simply Filter SMS is an iOS app (Swift/SwiftUI, iOS 16.6+) that filters unknown SMS messages using Apple's IdentityLookup framework. It includes a Message Filter Extension that classifies incoming messages as junk, transaction, or promotion, and a Reporting Extension that lets users report messages directly from iOS Messages. Data syncs across devices via CoreData + CloudKit (NSPersistentCloudKitContainer).

App Store: https://apps.apple.com/us/app/simply-filter-sms/id1603222959

## Build & Test

Open `Simply Filter SMS.xcodeproj` in Xcode. No package managers (SPM/CocoaPods) are used.

**Targets:**
- `Simply Filter SMS` — Main app
- `Message Filter Extension` — ILMessageFilterExtension (`.appex`)
- `Reporting Extension` — ILClassificationUIExtensionViewController (`.appex`) — user-initiated message reporting from iOS Messages
- `Tests` — Unit tests
- `UI Tests` — Snapshot tests via Fastlane

**Command-line build/test:**
```bash
xcodebuild -project "Simply Filter SMS.xcodeproj" -scheme "Simply Filter SMS" -destination 'platform=iOS Simulator,name=iPhone 16' build
xcodebuild -project "Simply Filter SMS.xcodeproj" -scheme "Simply Filter SMS" -destination 'platform=iOS Simulator,name=iPhone 16' test
```

**Localization:** BartyCrouch normalizes `.strings` files (English + Hebrew). Config in `.bartycrouch.toml`.

## Architecture

Three-layer clean architecture with protocol-based dependency injection:

### Framework Layer (`Simply Filter SMS/Framework Layer/`)
- **AppManager** — Singleton service locator (`AppManager.shared`). Creates and holds all managers/services. Use `AppManager(inMemory: true)` for previews/tests.
- **MessageEvaluationManager** — Core filtering engine. Evaluates sender + body against user filters, automatic rules, and language filters. Shared between app and extension.
- **PersistanceManager** — CoreData CRUD operations for `Filter`, `AutomaticFiltersRule`, `AutomaticFiltersLanguage` entities.
- **AutomaticFilterManager** — Fetches community filter lists from S3, applies automatic rules (block links, numbers-only senders, short senders, emails, emojis, all unknown).
- **DefaultsManager** — UserDefaults wrapper for app settings.
- **NetworkSyncManager** — NWPathMonitor + CloudKit sync status tracking.
- **TipJarManager** — StoreKit 2 in-app purchase manager for consumable tip jar products.

Every manager has a corresponding `*Protocol` in `Managers/Protocols/` for testability.

### Services Layer (`Simply Filter SMS/Services Layer/`)
- **AmazonS3Service** — Fetches automatic filter lists from AWS S3.
- **ReportMessageService** — Reports spam/ham to `https://api.ben-dahan.com/report` (public endpoint, no auth). Used by the in-app reporting UI. The Reporting Extension uses the same endpoint via iOS system delivery (`ILClassificationExtensionNetworkReportDestination`).
- **HTTPService** — Base class for HTTP requests with `URLRequestProtocol`.

### View Layer (`Simply Filter SMS/View Layer/`)
- **Screens/** — SwiftUI views: `AppHomeView`, `FilterListView`, `AddFilterView`, `TestFiltersView`, `LanguageListView`, `AboutView`, `HelpView`, `ReportMessageView`, `EnableExtensionVideoView`, `WhatsNewView`, `TipJarView`.
- **Others/** — `BaseViewModel` (base class for all ViewModels), reusable components, button styles, view modifiers.

### Shared with Extension (`Framework Layer/Shared with Extension/`)
Code compiled into both the app and the Message Filter Extension:
- `Constsants.swift` — All enums (`FilterType`, `DenyFolderType`, `FilterTarget`, `FilterMatching`, `FilterCase`, `RuleType`, `ReportType`) and global constants.
- `SharedExtensions.swift` — Extensions on `Filter` (CoreData), `NLLanguage`, `ILMessageFilterAction`, `String`, plus the `~` postfix operator.
- `AppPersistentCloudKitContainer.swift` — CoreData/CloudKit container setup with App Group (`group.com.grizz.apps.dev.simply-filter-sms`).

### Message Filter Extension (`Message Filter Extension/`)
- `MessageFilterExtension.swift` — Entry point implementing `ILMessageFilterQueryHandling`. Uses `MessageEvaluationManager` to evaluate messages offline (no network deferral).

## MVVM Pattern

Every screen follows the same structure:

- **Nested ViewModel:** Declared as `class ViewModel: BaseViewModel, ObservableObject` inside `extension SomeView { }`. Always accessed as `SomeView.ViewModel`.
- **BaseViewModel:** Holds a single `appManager: AppManagerProtocol` property (defaults to `AppManager.shared`). All ViewModels subclass it for DI access.
- **View owns ViewModel:** Most views use `@ObservedObject var model: ViewModel`. However, screens presented as sheets or that experienced re-rendering bugs use `@StateObject` with the `init(model:) { _model = StateObject(wrappedValue: model) }` pattern — this prevents SwiftUI from recreating the ViewModel on parent re-renders. Current `@StateObject` screens: `AboutView`, `CountryListView`, `ReportMessageView`, `TestFiltersView`, `TipJarView`. Views never access managers directly.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adibendahan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
