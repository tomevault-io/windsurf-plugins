---
trigger: always_on
description: Swift development: SwiftUI, Combine, async/await, iOS patterns, and Apple platform conventions
---


# Swift Development Patterns

Modern Swift patterns for iOS, macOS, and Apple platform development.

## Swift Workflow

Before changing Swift or Apple-platform code:

```text
1. Determine whether this is a Swift package or an Xcode-managed app
2. Read `Package.swift` or inspect the existing project structure first
3. For version-sensitive work, verify current Swift, SDK, or Xcode details with the actual current date
4. Never hand-edit IDE-managed project metadata
```

### ⚠️ CRITICAL: Xcode Project Files

**NEVER manually create or edit:**
- `*.xcodeproj/project.pbxproj` - This is a complex binary-like file managed by Xcode
- `*.xcworkspace/contents.xcworkspacedata`
- `*.xcodeproj/xcuserdata/*`

**These files MUST be created through:**
- Xcode IDE (Create New Project)
- `swift package init` for Swift Package Manager projects
- `xcodebuild` commands for CI/CD

**Why?** The `.pbxproj` file has:
- UUID references that must be consistent
- Specific formatting Xcode expects
- Build settings that are complex to replicate
- Manual creation will result in CORRUPTED projects

### CLI-First Swift Development

**For Swift Packages (preferred for libraries):**
```bash
# Create new package (NEVER manually create Package.swift)
swift package init --type library
swift package init --type executable
swift package init --type tool

# Add dependencies
# Edit Package.swift, then:
swift package resolve
swift build
swift test
```

**For iOS/macOS Apps:**
```bash
# You MUST use Xcode to create .xcodeproj
# There is NO CLI equivalent for full iOS projects

# Build from command line (project must exist)
xcodebuild -project MyApp.xcodeproj -scheme MyApp -sdk iphonesimulator build

# Run tests
xcodebuild test -project MyApp.xcodeproj -scheme MyApp -destination 'platform=iOS Simulator,name=iPhone 15'
```

### Post-Edit Verification

After meaningful Swift changes, run the smallest useful check for the task:

```bash
# Swift Package
swift build
swift test

# Xcode Project
xcodebuild -project MyApp.xcodeproj -scheme MyApp -sdk iphonesimulator build
```

Add lint or formatting checks when the repo already uses them or the change is substantial.

### Common Swift Syntax Traps (Avoid These!)

```swift
// WRONG: Force unwrapping optionals
let name = user.name!  // Crashes if nil!

// CORRECT: Safe unwrapping
guard let name = user.name else {
    return
}

// Or use optional chaining
let name = user.name ?? "Unknown"

// WRONG: Strong reference cycles
class Parent {
    var child: Child?
}
class Child {
    var parent: Parent?  // Creates retain cycle!
}

// CORRECT: Use weak or unowned
class Child {
    weak var parent: Parent?
}

// WRONG: Blocking main thread
func loadData() {
    let data = URLSession.shared.data(from: url)  // Blocks UI!
}

// CORRECT: Use async/await
func loadData() async throws -> Data {
    let (data, _) = try await URLSession.shared.data(from: url)
    return data
}

// WRONG: Not using @MainActor for UI updates
class ViewModel: ObservableObject {
    @Published var items: [Item] = []  // May update from background!
}

// CORRECT: Use @MainActor
@MainActor
class ViewModel: ObservableObject {
    @Published var items: [Item] = []  // Guaranteed main thread
}
```

---

## SwiftUI Fundamentals

### View Structure
```swift
struct ContentView: View {
    // State at the top
    @State private var isLoading = false
    @State private var items: [Item] = []
    
    // Environment and observed objects
    @EnvironmentObject var appState: AppState
    @ObservedObject var viewModel: ContentViewModel
    
    // Computed properties
    private var filteredItems: [Item] {
        items.filter { $0.isActive }
    }
    
    var body: some View {
        NavigationStack {
            List(filteredItems) { item in
                ItemRow(item: item)
            }
            .navigationTitle("Items")
            .toolbar {
                ToolbarItem(placement: .primaryAction) {
                    Button("Add", action: addItem)
                }
            }
        }
        .task {
            await loadItems()
        }
    }
    
    // Actions at the bottom
    private func addItem() { ... }
    private func loadItems() async { ... }
}
```

### Property Wrappers

```swift
// @State - owned by the view, simple value types
@State private var count = 0
@State private var text = ""

// @Binding - reference to parent's state
struct ChildView: View {
    @Binding var isPresented: Bool
}

// @StateObject - owned by the view, reference types (create once)
@StateObject private var viewModel = ViewModel()

// @ObservedObject - reference from parent (don't create)
@ObservedObject var viewModel: ViewModel

// @EnvironmentObject - shared app-wide state
@EnvironmentObject var appState: AppState

// @Environment - system values
@Environment(\.colorScheme) var colorScheme
@Environment(\.dismiss) var dismiss

// @AppStorage - UserDefaults persistence
@AppStorage("username") var username = ""
```

### View Modifiers
```swift
struct ContentView: View {
    var body: some View {
        Text("Hello")
            .font(.headline)
            .foregroundColor(.primary)
            .padding()
            .background(.regularMaterial)
            .cornerRadius(12)
            .shadow(radius: 4)
    }
}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/advance-minimax-m2-cursor-rules](https://github.com/madebyaris/advance-minimax-m2-cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
