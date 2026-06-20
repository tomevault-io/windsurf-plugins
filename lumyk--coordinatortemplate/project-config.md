---
trigger: always_on
description: Generate a new iOS app project with Coordinator + MVVM + DI architecture. Creates full project skeleton with navigation, networking, auth, and DI container.
---


# ios-coordinator

Generate a complete iOS project skeleton using Coordinator + MVVM + DI architecture.

## Usage

`$ARGUMENTS` is the **project name**. Use it as:
- Root directory name
- Xcode target name and sources directory name
- Swift module name (struct name for `@main` App: `${ARGUMENTS}App`)
- Bundle identifier prefix: `com.$ARGUMENTS_LOWER` (lowercased)
- UserDefaults suite key: `com.$ARGUMENTS_LOWER.user-defaults` (lowercased)
- File headers use `$ARGUMENTS` as the project name
- Copyright: `Copyright © Yevhenii Kalashnikov. All rights reserved.`

If `$ARGUMENTS` is empty, ask the user for a project name before proceeding.

Replace all occurrences of `$ARGUMENTS` in the generated code with the actual project name. Replace `$ARGUMENTS_LOWER` with the lowercased version.

---

## Step 1: Create Directory Structure

```
$ARGUMENTS/
├── project.yml
├── Packages/
│   └── Coordinator/
│       ├── Package.swift
│       └── Sources/
│           ├── Coordinator.swift
│           ├── CoordinatorRoot.swift
│           ├── CoordinatorBranch.swift
│           └── Destination.swift
└── $ARGUMENTS/
    ├── ${ARGUMENTS}App.swift
    ├── Core/
    │   ├── Context.swift
    │   ├── ServiceProvider.swift
    │   ├── Session/
    │   │   ├── Session.swift
    │   │   └── SessionProvider.swift
    │   ├── Networking/
    │   │   ├── NetworkingService.swift
    │   │   ├── AnyMutation.swift
    │   │   ├── NetworkingServiceError.swift
    │   │   └── HttpMethod.swift
    │   ├── Helpers/
    │   │   ├── Keychain.swift
    │   │   ├── UserDefault.swift
    │   │   ├── UserDefaults+.swift
    │   │   ├── DataCache.swift
    │   │   ├── Codable+.swift
    │   │   ├── Error+.swift
    │   │   └── URL+.swift
    │   ├── Services/
    │   │   └── UserSettings.swift
    │   └── Models/
    │       ├── User.swift
    │       ├── APISession.swift
    │       ├── APIError.swift
    │       └── Mutations.swift
    └── UI/
        └── Interfaces/
            ├── MainView.swift
            ├── Tabs/
            │   └── TabsView.swift
            ├── Auth/
            │   ├── Login/
            │   │   ├── LoginView.swift
            │   │   ├── LoginViewModel.swift
            │   │   └── LoginViewBuilder.swift
            │   └── SignUp/
            │       ├── SignUpView.swift
            │       ├── SignUpViewModel.swift
            │       └── SignUpViewBuilder.swift
            ├── Home/
            │   ├── HomeView.swift
            │   ├── HomeViewModel.swift
            │   └── HomeViewBuilder.swift
            ├── Detail/
            │   ├── DetailView.swift
            │   ├── DetailViewModel.swift
            │   └── DetailViewBuilder.swift
            ├── Profile/
            │   ├── ProfileView.swift
            │   ├── ProfileViewModel.swift
            │   └── ProfileViewBuilder.swift
            └── Settings/
                ├── SettingsView.swift
                ├── SettingsViewModel.swift
                └── SettingsViewBuilder.swift
```

---

## Step 2: Generate Files

Every Swift file starts with this header:
```swift
//
//  FileName.swift
//  $ARGUMENTS
//
//  Copyright © Yevhenii Kalashnikov. All rights reserved.
//
```

Generate each file below with **exact** content. Replace `$ARGUMENTS` with the project name.

---

### 2.1 Coordinator SPM Package

#### `Packages/Coordinator/Package.swift`

```swift
// swift-tools-version: 6.0
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
    name: "Coordinator",
    platforms: [.iOS(.v17)],
    products: [
        .library(
            name: "Coordinator",
            targets: ["Coordinator"]
        ),
    ],
    dependencies: [
        // Dependencies can be added here if needed
    ],
    targets: [
        .target(
            name: "Coordinator",
            dependencies: [
                // Dependencies can be added here if needed
            ],
            path: "Sources",
            resources: [
                // Resources can be added here if needed
            ]
        ),
    ]
)
```

#### `Packages/Coordinator/Sources/Coordinator.swift`

```swift
import SwiftUI

@MainActor
final public class Coordinator<Context: Sendable>: ObservableObject {
    public let context: Context
    private weak var rootCoordinator: Coordinator?

    @Published public var path = NavigationPath()
    @Published public var sheet: Destination<Context>?
    @Published public var fullScreenCover: Destination<Context>?

    public init(context: Context) {
        self.context = context
    }

    public init(rootCoordinator: Coordinator) {
        context = rootCoordinator.context
        self.rootCoordinator = rootCoordinator
    }

    public func push(_ destination: Destination<Context>) {
        path.append(destination)
    }

    public func present(sheet destination: Destination<Context>) {
        sheet = destination
    }

    public func present(fullScreenCover destination: Destination<Context>) {
        fullScreenCover = destination
    }

    public func pop() {
        guard !path.isEmpty else { return }
        path.removeLast()
    }

    public func popToRoot() {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lumyk/CoordinatorTemplate](https://github.com/Lumyk/CoordinatorTemplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
