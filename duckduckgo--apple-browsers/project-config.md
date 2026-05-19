---
trigger: always_on
description: ALWAYS use the centralized dependency provider for all service dependencies:
---


# iOS DuckDuckGo Browser Architecture Rules

## Dependency Injection and AppDependencies

### Use AppDependencyProvider Pattern
ALWAYS use the centralized dependency provider for all service dependencies:

```swift
// ✅ CORRECT - Use the shared dependency provider
final class FeatureViewModel: ObservableObject {
    private let networkService: NetworkServiceProtocol
    
    init(dependencies: DependencyProvider = AppDependencyProvider.shared) {
        self.networkService = dependencies.networkService
    }
}

// ❌ INCORRECT - Direct singleton access
final class FeatureViewModel: ObservableObject {
    private let networkService = NetworkService.shared // Avoid singletons
}
```

### Protocol-Based Dependencies
ALWAYS define protocols for dependencies to enable testing:

```swift
// ✅ CORRECT - Protocol abstraction
protocol FeatureServiceProtocol {
    func fetchData() async throws -> [Item]
}

final class FeatureService: FeatureServiceProtocol {
    // Implementation
}

// ❌ INCORRECT - Concrete dependency
final class ViewModel {
    private let service: FeatureService // Hard to test
}
```

## AppSettings and Configuration

### Use AppSettings Protocol
ALWAYS access settings through the AppSettings protocol:

```swift
// ✅ CORRECT - Protocol-based settings access
final class SettingsViewModel: ObservableObject {
    private let appSettings: AppSettings
    
    init(appSettings: AppSettings) {
        self.appSettings = appSettings
    }
    
    var isFeatureEnabled: Bool {
        get { appSettings.featureEnabled }
        set { appSettings.featureEnabled = newValue }
    }
}

// ❌ INCORRECT - Direct UserDefaults access
final class SettingsViewModel: ObservableObject {
    var isFeatureEnabled: Bool {
        get { UserDefaults.standard.bool(forKey: "feature_enabled") }
        set { UserDefaults.standard.set(newValue, forKey: "feature_enabled") }
    }
}
```

### UserDefaults Property Wrapper
Use the established @UserDefaultsWrapper pattern for new settings:

```swift
// ✅ CORRECT - Property wrapper usage
extension AppUserDefaults {
    @UserDefaultsWrapper(key: .newFeatureEnabled, defaultValue: false)
    var newFeatureEnabled: Bool
}

// ❌ INCORRECT - Manual UserDefaults handling
extension AppUserDefaults {
    var newFeatureEnabled: Bool {
        get { userDefaults.bool(forKey: "new_feature_enabled") }
        set { userDefaults.set(newValue, forKey: "new_feature_enabled") }
    }
}
```

## Navigation and Coordinators

### Use MainCoordinator for App-Level Navigation
ALWAYS use MainCoordinator for deep links, URL handling, and app-level navigation:

```swift
// ✅ CORRECT - MainCoordinator usage
@MainActor
final class FeatureCoordinator {
    private weak var mainCoordinator: MainCoordinator?
    
    func handleFeatureAction() {
        mainCoordinator?.handleURL(featureURL)
    }
}

// ❌ INCORRECT - Direct navigation from ViewModels
final class FeatureViewModel: ObservableObject {
    func handleAction() {
        // Don't navigate directly from ViewModels
        navigationController?.pushViewController(detailVC, animated: true)
    }
}
```

### URL Handling Pattern
Implement URLHandling protocol for custom URL schemes:

```swift
// ✅ CORRECT - URLHandling protocol implementation
extension FeatureCoordinator: URLHandling {
    func handleURL(_ url: URL) {
        guard url.scheme == "duckduckgo",
              url.host == "feature" else { return }
        
        presentFeature(with: url.queryParameters)
    }
    
    func shouldProcessDeepLink(_ url: URL) -> Bool {
        return url.scheme == "duckduckgo" && url.host == "feature"
    }
}
```

## SwiftUI and Design System Integration

### Use DesignResourcesKit Colors
ALWAYS use semantic colors from DesignResourcesKit:

```swift
// ✅ CORRECT - Semantic color usage
struct FeatureView: View {
    var body: some View {
        VStack {
            Text("Title")
                .foregroundColor(Color(designSystemColor: .textPrimary))
            
            Rectangle()
                .fill(Color(designSystemColor: .surface))
        }
        .background(Color(designSystemColor: .background))
    }
}

// ❌ INCORRECT - Hardcoded colors
struct FeatureView: View {
    var body: some View {
        VStack {
            Text("Title")
                .foregroundColor(.black) // Don't hardcode colors
            
            Rectangle()
                .fill(.gray) // Use semantic colors instead
        }
    }
}
```

### Use DesignResourcesKit Icons
ALWAYS use icons from DesignResourcesKitIcons:

```swift
// ✅ CORRECT - Design system icons
struct IconButton: View {
    var body: some View {
        Button(action: action) {
            Image(uiImage: DesignSystemImages.Glyphs.Size16.add)
                .foregroundColor(Color(designSystemColor: .accent))
        }
    }
}

// ❌ INCORRECT - System icons or custom images
struct IconButton: View {
    var body: some View {
        Button(action: action) {
            Image(systemName: "plus") // Use design system icons
        }
    }
}
```

### Theme Integration
Use Theme protocol for complex color requirements:

```swift
// ✅ CORRECT - Theme integration
struct ThemedView: View {
    @EnvironmentObject var themeManager: ThemeManager
    
    var body: some View {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
