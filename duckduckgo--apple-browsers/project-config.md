---
trigger: always_on
description: *This style guide is based on the [official iOS style guide](iOS/styleguide/STYLEGUIDE.md) and incorporates DuckDuckGo-specific patterns and requirements.*
---


# Swift Code Style Guide

*This style guide is based on the [official iOS style guide](iOS/styleguide/STYLEGUIDE.md) and incorporates DuckDuckGo-specific patterns and requirements.*

## Correctness

**Strive to make your code compile without warnings.** This rule informs many style decisions such as using `#selector` types instead of string literals.

## SwiftLint

We use [SwiftLint](https://github.com/realm/SwiftLint) for enforcing Swift style and conventions. See the [SwiftLint configuration](.swiftlint.yml) for specific rules.

**Key SwiftLint settings**:
- Line length: 150 characters (not the default 100)
- Force cast/try: warnings (not errors for pragmatic development)
- Identifier naming: flexible for single-letter variables in closures

## Naming Conventions

Follow the [Swift API Design Guidelines](https://swift.org/documentation/api-design-guidelines/) with these key principles:

### Core Principles
- **Clarity at the call site** over brevity
- **Use camelCase** (not snake_case)
- **UpperCamelCase** for types and protocols
- **lowerCamelCase** for everything else
- **Include all needed words** while omitting needless words
- **Use names based on roles**, not types

### Type Names
```swift
// ✅ CORRECT: Descriptive, UpperCamelCase
class UserAuthenticationManager { }
struct BookmarkItem { }
enum NavigationState { }
protocol DataSourceProtocol { }

// ❌ INCORRECT: Too generic
class Manager { }
struct Data { }
```

### Variable and Function Names
```swift
// ✅ CORRECT: Descriptive lowerCamelCase
let maximumRetryCount = 3
var isLoading = false
func fetchUserData() { }

// Boolean properties should read like assertions
var isEnabled: Bool
var hasCompleted: Bool
var canDelete: Bool

// ❌ INCORRECT: Abbreviations and unclear names
let usrMgr = UserManager()
func calcTotal() { }
```

### Protocol Naming
```swift
// ✅ CORRECT: Capability protocols end in -able, -ible, -ing
protocol Loadable { }
protocol Refreshable { }
protocol UserAuthenticating { }

// ✅ CORRECT: Type protocols are nouns
protocol DataSource { }
protocol Delegate { }
```

### Method Naming Patterns
```swift
// ✅ CORRECT: Method naming patterns
// Factory methods begin with "make"
func makeLocationManager() -> CLLocationManager

// Verb methods follow -ed, -ing rule for non-mutating
func sorted() -> [Element]  // non-mutating
func sort()                 // mutating

// Boolean methods read like assertions
func canDelete() -> Bool
func hasCompleted() -> Bool
```

### Delegate Methods
When creating custom delegate methods, the **unnamed first parameter should be the delegate source**:

```swift
// ✅ CORRECT: Delegate pattern
func namePickerView(_ namePickerView: NamePickerView, didSelectName name: String)
func namePickerViewShouldReload(_ namePickerView: NamePickerView) -> Bool

// ❌ INCORRECT: Missing source parameter
func didSelectName(namePicker: NamePickerViewController, name: String)
func namePickerShouldReload() -> Bool
```

### Use Type Inferred Context
Use compiler inferred context to write shorter, clear code:

```swift
// ✅ CORRECT: Type inferred context
let selector = #selector(viewDidLoad)
view.backgroundColor = .red
let toView = context.view(forKey: .to)
let view = UIView(frame: .zero)

// ❌ INCORRECT: Redundant type information
let selector = #selector(ViewController.viewDidLoad)
view.backgroundColor = UIColor.red
let toView = context.view(forKey: UITransitionContextViewKey.to)
let view = UIView(frame: CGRect.zero)
```

### Generics
Generic type parameters should be **descriptive, UpperCamelCase names**:

```swift
// ✅ CORRECT: Descriptive generic names
struct Stack<Element> { ... }
func write<Target: OutputStream>(to target: inout Target)
func swap<T>(_ a: inout T, _ b: inout T)  // T is acceptable when no meaningful relationship

// ❌ INCORRECT: Non-descriptive or wrong case
struct Stack<T> { ... }
func write<target: OutputStream>(to target: inout target)
```

### Language
Use **US English spelling** to match Apple's API:

```swift
// ✅ CORRECT: US English
let color = "red"

// ❌ INCORRECT: British English
let colour = "red"
```

## Code Organization

### File Structure
```swift
// 1. Import statements (minimal - only what's needed)
import UIKit
import Combine

// 2. Protocol definitions
protocol FeatureDelegate: AnyObject {
    func featureDidUpdate()
}

// 3. Main type declaration
class FeatureViewController: UIViewController {
    // Properties first
    private let viewModel: FeatureViewModel
    
    // Lifecycle methods
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
    }
    
    // Private methods
    private func setupUI() { }
}

// 4. Extensions for protocol conformance
// MARK: - UITableViewDataSource
extension FeatureViewController: UITableViewDataSource {
    // Protocol methods
}
```

### Protocol Conformance
**Prefer separate extensions** for protocol conformance to keep related methods grouped:

```swift
// ✅ CORRECT: Separate extensions
class MyViewController: UIViewController {
    // class implementation
}

// MARK: - UITableViewDataSource
extension MyViewController: UITableViewDataSource {
    // table view data source methods
}

// MARK: - UIScrollViewDelegate

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
