---
trigger: always_on
description: Swift and SwiftUI coding standards and style guidelines
---


# Swift Style Guide

Swift and SwiftUI coding standards and style guidelines for our iOS codebase. These guidelines ensure consistency, maintainability, and high code quality across Swift projects.

## Core Principles

- **KISS (Keep It Simple, Stupid)** - Always choose the simplest, most maintainable solution
- **SwiftUI First** - Always prefer SwiftUI over UIKit when possible
- **Less is More** - Always avoid unnecessary complexity, the best code is no code
- **Self-Documenting** - Always make code obvious and clear without comments

## Linter Errors and False Positives

### Ignoring False Positive Errors

**Always ignore false positive linter errors from Cursor/VS Code** when working with Swift code. The Swift language server in VS Code/Cursor cannot properly resolve Swift symbols and dependencies that are correctly configured in Xcode.

**Common false positive errors to ignore:**

- `Cannot find 'X' in scope` - When X is clearly defined in the codebase
- `Reference to member 'X' cannot be resolved without a contextual type` - When X is a valid SwiftUI/UIKit type
- `No such module 'X'` - When the module is correctly imported in Xcode
- `Value of type 'X' has no member 'Y'` - When Y is a valid member in Xcode
- Any reference errors for custom views, models, or utilities that compile successfully in Xcode

**Workflow:**

- Use Cursor for AI assistance and code generation
- Use Xcode for actual development and compilation
- Only report real compilation errors from Xcode, not linter errors from Cursor/VS Code
- Trust Xcode's build system over Cursor's language server for Swift

## File Organization

### Directory Structure

- Always organize code in a predictable and scalable way
- Always keep related code close together
- Always use clear, descriptive directory names
- Always follow consistent patterns across the project
- Always use singular for categories/domains (e.g. `Feature/`, `Model/`, `View/`)
- Always use plural for collections/lists (e.g. `Features/`, `Models/`, `Views/`)

✅ Good:

```swift
Tapling/
  Features/
    Collectible/
      Environment/
        Models/
        CollectibleRegistry.swift
      Views/
  Routes/
    Settings/
      SettingsView.swift
  Views/
    ActionRowView.swift
```

❌ Bad:

```swift
Tapling/
  feature/           // Wrong: Should be Features (plural)
    collectible/     // Wrong: Should be Collectible (PascalCase)
      model/         // Wrong: Should be Models (plural)
```

### File Naming

- Always use `PascalCase` for Swift files (e.g. `SettingsView.swift`, `TaplingSettings.swift`)
- Always match the file name to the primary type/struct/class it contains
- Always use descriptive names that indicate purpose

✅ Good:

```swift
SettingsView.swift        // Contains SettingsView struct
TaplingSettings.swift     // Contains TaplingSettings class
ActionRowView.swift       // Contains ActionRowView struct
```

❌ Bad:

```swift
settings.swift           // Wrong: Should be PascalCase
Settings.swift           // Wrong: Too generic
View.swift               // Wrong: Not descriptive
```

## SwiftUI View Structure

### View Organization

- Always follow the 3-layer structure: Variables → UI → Actions
- Always use `// MARK: - UI` to separate UI components from actions
- Always use `// MARK: - Actions` to separate actions from UI
- Never add more than these 2 MARKs - if you need more organization, split the view

✅ Good:

```swift
struct SettingsView: View {
    // Variables (no MARK needed)
    @State private var isEnabled = false
    @QuerySingleton private var settings: Settings

    // MARK: - UI

    var body: some View {
        Form {
            headerSection
        }
    }

    private var headerSection: some View {
        Section {
            Toggle("Enabled", isOn: isEnabledBinding)
        }
    }

    // MARK: - Actions

    private func saveSettings() {
        try? modelContext.save()
    }
}
```

❌ Bad:

```swift
struct SettingsView: View {
    // MARK: - Properties  // Wrong: No MARK for variables
    @State private var isEnabled = false

    // MARK: - UI
    var body: some View { }

    // MARK: - Search UI     // Wrong: Too many MARKs
    // MARK: - Filter UI     // Wrong: Split into separate views instead
    // MARK: - Actions
}
```

### View Components

- Always use computed properties for view sections
- Always use descriptive names ending with `Section`, `View`, or `Button` (e.g. `headerSection`, `settingsForm`, `saveButton`)
- Always keep view components private unless they need to be reused elsewhere

✅ Good:

```swift
private var headerSection: some View {
    VStack {
        Text("Title")
    }
}

private var settingsForm: some View {
    Form {
        // ...
    }
}
```

❌ Bad:

```swift
var header: some View { }        // Wrong: Not descriptive
public var section: some View { } // Wrong: Should be private unless reused
```

## Naming Conventions

### Types and Structures

- Always use `PascalCase` for types, structs, classes, enums, protocols
- Always use descriptive names that clearly indicate purpose
- Always prefix protocols with descriptive names (e.g. `SingletonModel`, not `Model`)

✅ Good:

```swift
struct SettingsView: View { }
class DataContainer { }
protocol SingletonModel { }
enum Rarity { }
```

❌ Bad:

```swift

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [builder-group/focuscat](https://github.com/builder-group/focuscat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
