---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with
---

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

# Project description

We are creating the next generation of automation through an AI agents execution
environment. We believe that instead of complex programming, users simply
describe what they want accomplished, and our platform handles the rest. A
declarative approach. Anyone can create AI agents by connecting their existing
applications via tools and assembling a workflow. These agents run automatically
based on schedules or triggers, and can be easily shared. Our iOS app makes it
seamless to share content with your agents and trigger workflows on-the-go. We
serve both consumers looking to automate personal tasks and businesses needing
regular automation. It's a platform designed for rapid experimentation—launch
small automation ideas, see what works, and iterate quickly.

# Build & Development Commands

## Project Setup

```bash
# Initial setup (installs SwiftLint and pre-commit hooks)
make setup

# Install dependencies 
tuist install

# Generate Xcode project files with Tuist
tuist generate

# Open the project in Xcode
tuist edit
```

### Build Commands

```bash
# Build the project
tuist build

# Clean build artifacts
tuist clean

# Warm cache for faster builds
tuist cache
```

### Code Quality

```bash
# Run SwiftLint
swiftlint ./Bluerage/ --fix --strict

# Analyze with SwiftLint (includes more thorough checks)
swiftlint analyze ./Bluerage/
```

## Testing

```bash
# Run tests
tuist test
```

# Coding Style Guide

## General Principles

- **Explicit over implicit**: Always prefer explicit code that clearly shows
  intent
- **Single responsibility**: Each class/struct should have one clear purpose
- **Dependency injection**: Use FactoryKit (Swift package) for DI, never global
  singletons
- **Error handling**: Prefer `do-catch` over returning `nil`, never make
  throwing methods that return optionals
- **Async patterns**: Use Swift concurrency over GCD/Combine when possible
- **Always use explicit `self.`** in all method calls and property access

### File Naming Conventions

- **UIKit ViewControllers**: `{Name}Controller` (e.g., `WelcomeController`)
- **SwiftUI Views**: `{Name}View` (e.g., `LoginView`)
- **ViewModels**: `{Name}ViewModel` (e.g., `LoginViewModel`)
- **Extensions**: `{Target}+{Purpose}.swift` (e.g., `PlaceholderView+Ext.swift`)
- **Dependencies**: `{ServiceName}+Dependency.swift`
- **Use camelCase for all Swift/Objective-C naming**

### Project Structure

```
Sources/
├── Core/                    # Shared code, DI, extensions
├── Kit/               # Services, single source of truth
├── Generated/              # OpenAPI generations
└── UI/
    ├── Components/         # Reusable UI components
    └── Screens/           # Individual screens
Resources/                  # Assets, strings, etc.
```

### Code Organization Within Files

**Order of declarations:**

1. `open static` declarations
2. `open` declarations
3. `public static` declarations
4. `public` declarations
5. `internal static` declarations
6. `internal` declarations
7. `private static` declarations
8. `private` declarations

**UIKit ViewController structure:**

```swift
final class ExampleController: UIViewController {

    // MARK: - Properties
    
    private let containerView = UIView(frame: .zero)
    
    @Injected(\.authSession) private var authSession

    // MARK: - Lifecycle
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        self.view.addSubview(self.containerView)
        
        self.setUpSelf()
        self.setUpContainerView()
    }
    
    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()
        
        self.containerView.pin
            .all()
    }

    // MARK: - Setup
    
    private func setUpSelf() {
        self.view.backgroundColor = UIColor.res.black
    }
    
    private func setUpContainerView() {
        // View setup code
    }

    // MARK: - Actions
    
    private func didTapButton() {
        // Action handling
    }

}
```

### Swift Coding Patterns

**Always use explicit `self.`:**

```swift
// ✅ Preferred
self.view.addSubview(self.containerView)
self.setUpViews()

// ❌ Avoid
view.addSubview(containerView)
setUpViews()
```

**Prefer guard statements over if-let:**

```swift
// ✅ Preferred
guard let userId = self.authSession.state.userId else {
    return
}

// ❌ Avoid
if let userId = self.authSession.state.userId {
    // continue with logic
}
```

**Use do-catch for error handling:**

```swift
// ✅ Preferred
do {
    let result = try await service.fetchData()
    // handle result
} catch {
    Logger.domain.error("Error fetching data: \(error)")
}

// ❌ Never do this
func fetchData() throws -> Result? {
    // Don't make throwing methods return optionals
}
```

### SwiftUI Architecture & Patterns

**Use MVVM + State with Observation:**

- NEVER use ObservableObject

```swift
@MainActor
@Observable
final class ExampleViewModel {
    enum State {
        case skeleton
        case loaded(data: [Item])
        case empty
        case error(Error)
    }
    
    private(set) var state: State = .skeleton
    
    @ObservationIgnored
    @Injected(\.dataService) private var dataService
    

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blueragesoftware/iOS](https://github.com/blueragesoftware/iOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
