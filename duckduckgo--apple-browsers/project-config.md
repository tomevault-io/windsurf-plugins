---
trigger: always_on
description: **Example:** See [singleton-antipattern.swift](anti-patterns/singleton-antipattern.swift)
---


# Anti-patterns and Common Mistakes to Avoid

## Singleton Anti-patterns

### ❌ NEVER: Static Shared Instances Without Dependency Injection (.shared instance pattern)
**Example:** See [singleton-antipattern.swift](anti-patterns/singleton-antipattern.swift)

### ❌ NEVER: Global State Access
```swift
// ❌ AVOID - Global state access
var globalSettings: [String: Any] = [:]

func someFunction() {
    globalSettings["key"] = "value" // Global state is hard to test and debug
}

// ✅ CORRECT - Injected dependencies
final class SomeService {
    private let settings: AppSettings
    
    init(settings: AppSettings) {
        self.settings = settings
    }
    
    func someFunction() {
        settings.setValue("value", for: "key")
    }
}
```

## Async/Await Anti-patterns

### ❌ NEVER: UI Updates Without @MainActor
**Example:** See [async-ui-updates.swift](anti-patterns/async-ui-updates.swift)

### ❌ NEVER: Unhandled Async Errors
```swift
// ❌ AVOID - Swallowing async errors
func fetchData() async {
    let data = try? await networkService.getData() // Silently ignoring errors
    // Process data...
}

// ✅ CORRECT - Proper error handling
func fetchData() async throws {
    let data = try await networkService.getData()
    // Process data...
}

// Or handle errors appropriately:
func fetchData() async {
    do {
        let data = try await networkService.getData()
        // Process data...
    } catch {
        // Log error and show user-friendly message
        logger.error("Failed to fetch data: \(error)")
        await showError(error)
    }
}
```

### ❌ NEVER: Blocking Main Thread with Sync Operations
```swift
// ❌ AVOID - Blocking main thread
@MainActor
func loadData() {
    let data = NetworkService.fetchDataSynchronously() // Blocks UI
    updateUI(with: data)
}

// ✅ CORRECT - Use async operations
@MainActor
func loadData() async {
    let data = try await NetworkService.fetchData() // Non-blocking
    updateUI(with: data)
}
```

## Memory Management Anti-patterns

### ❌ NEVER: Strong Reference Cycles in Closures
**Example:** See [memory-leak-closure.swift](anti-patterns/memory-leak-closure.swift)

### ❌ NEVER: Retaining View Controllers in Cache
```swift
// ❌ AVOID - Caching view controllers without cleanup
class NavigationManager {
    private var cachedViewControllers: [String: UIViewController] = [:]
    
    func getViewController(for identifier: String) -> UIViewController {
        if let cached = cachedViewControllers[identifier] {
            return cached // May contain stale data and strong references
        }
        let vc = createViewController(for: identifier)
        cachedViewControllers[identifier] = vc
        return vc
    }
}

// ✅ CORRECT - Cache view models, not view controllers
class NavigationManager {
    private var cachedViewModels: [String: ViewModel] = [:]
    
    func getViewController(for identifier: String) -> UIViewController {
        let viewModel = getOrCreateViewModel(for: identifier)
        return createViewController(with: viewModel)
    }
    
    private func getOrCreateViewModel(for identifier: String) -> ViewModel {
        if let cached = cachedViewModels[identifier] {
            return cached
        }
        let viewModel = createViewModel(for: identifier)
        cachedViewModels[identifier] = viewModel
        return viewModel
    }
}
```

## Error Handling Anti-patterns

### ❌ NEVER: Force Unwrapping Without Justification
**Example:** See [force-unwrapping.swift](anti-patterns/force-unwrapping.swift)

### ❌ NEVER: Generic Error Messages
```swift
// ❌ AVOID - Generic error handling
func handleError(_ error: Error) {
    print("Something went wrong") // Not helpful for debugging
    showAlert("Error occurred")   // Not helpful for users
}

// ✅ CORRECT - Specific error handling
enum NetworkError: LocalizedError {
    case noConnection
    case timeout
    case unauthorized
    case serverError(Int)
    
    var errorDescription: String? {
        switch self {
        case .noConnection:
            return "No internet connection. Please check your network settings."
        case .timeout:
            return "Request timed out. Please try again."
        case .unauthorized:
            return "You are not authorized to access this resource."
        case .serverError(let code):
            return "Server error (\(code)). Please try again later."
        }
    }
}

func handleNetworkError(_ error: NetworkError) {
    logger.error("Network error: \(error)")
    showAlert(error.localizedDescription)
}
```

## SwiftUI Anti-patterns

### ❌ NEVER: Heavy Computation in View Body
```swift
// ❌ AVOID - Expensive operations in body
struct ContentView: View {
    let items: [Item]
    
    var body: some View {
        List {
            ForEach(items) { item in
                Text(expensiveProcessing(item)) // Computed every view update
            }
        }
    }
    
    private func expensiveProcessing(_ item: Item) -> String {
        // Heavy computation
        return item.data.complexProcessing()
    }
}

// ✅ CORRECT - Pre-compute or use lazy loading
struct ContentView: View {
    @StateObject private var viewModel: ContentViewModel
    
    var body: some View {
        List {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
