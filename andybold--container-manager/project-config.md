---
trigger: always_on
description: This is a native macOS container management application built with SwiftUI. It supports Apple's container tool and provides both menu bar and desktop app modes.
---

# Claude Code Instructions for Container Manager

## Project Overview

This is a native macOS container management application built with SwiftUI. It supports Apple's container tool and provides both menu bar and desktop app modes.

## Code Style & Conventions

### SwiftUI Best Practices

#### Label Wrapping Prevention ⚠️ CRITICAL

**Always prevent label wrapping in horizontal layouts:**

When using SwiftUI controls with labels (Picker, Toggle, Button, etc.) in tight horizontal layouts, apply these modifiers:

```swift
// Segmented Picker Pattern
Picker("Label Text", selection: $binding) {
    ForEach(options) { option in
        Text(option).tag(option)
    }
}
.pickerStyle(.segmented)
.labelsHidden()  // Hide built-in label to prevent wrapping
.fixedSize()     // Prevent control compression
.frame(width: XXX) // Set explicit width

// Toggle Pattern
Toggle("Label Text", isOn: $binding)
    .toggleStyle(.switch)
    .fixedSize()  // Prevent label wrapping
    .help("Tooltip text")
```

**When to apply:**
- ✅ Segmented pickers in toolbars
- ✅ Toggles in horizontal layouts
- ✅ Any control in constrained horizontal space
- ✅ Controls where label might wrap to multiple lines

**Common locations:**
- Toolbar controls
- Inspector panels
- Settings views
- Modal headers

### Thread Safety & Performance

#### Background Processing

**Always run blocking operations on background threads:**

```swift
// ✅ Good - Non-blocking process execution
private func executeCommand() async -> String? {
    return await withCheckedContinuation { continuation in
        Task.detached {
            let process = Process()
            // ... setup process ...
            process.run()
            process.waitUntilExit()  // Blocks background thread only
            continuation.resume(returning: result)
        }
    }
}

// ❌ Bad - Blocks main thread
private func executeCommand() async -> String? {
    let process = Process()
    process.run()
    process.waitUntilExit()  // Blocks current thread!
    return result
}
```

**Thread-safe data access:**

```swift
// Use NSLock for simple synchronization
private let lock = NSLock()
nonisolated(unsafe) private var sharedData: [String: Data] = [:]

func updateData() {
    lock.lock()
    defer { lock.unlock() }
    // Safely access sharedData
}
```

### Naming Conventions

- **Views**: PascalCase (e.g., `ContainerListView`, `StatsView`)
- **Properties**: camelCase (e.g., `containerMonitor`, `isLoading`)
- **Methods**: camelCase with verb prefix (e.g., `startContainer`, `fetchLogs`)
- **Constants**: camelCase (e.g., `maxRetries`, `defaultTimeout`)

### File Organization

```
container-manager/
├── container-manager/           # Main app target
│   ├── *App.swift              # App entry point
│   ├── *SystemMonitor.swift    # Core business logic
│   ├── ContentView.swift       # Menu bar view
│   └── Assets.xcassets
├── Views/                       # UI components (root level)
│   ├── ContainerListView.swift
│   ├── StatsView.swift
│   └── ...
├── Tests/
└── docs/                        # All documentation
```

## Testing

### When to Write Tests

- New business logic in `ContainerSystemMonitor`
- Data models and parsers
- Network/Volume management features
- Complex algorithms (stats calculations, parsing)

### Testing Framework

Use the Testing framework (not XCTest) for new tests:

```swift
import Testing

@Test("Container stats parsing works correctly")
func testStatsParser() async throws {
    // Test implementation
}
```

## Git Commit Messages

Follow conventional commits format:

```
feat: implement stats collection for Apple container tool
fix: prevent label wrapping in stats view picker
docs: update architecture documentation
refactor: extract stats parsing to separate method
```

**Always include Co-Authored-By for AI assistance:**

```
feat: add detailed stats visualization

- Add enhanced charts with gradients
- Implement network I/O legend
- Add block I/O chart

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

## Common Patterns

### Animation Preferences

**Always respect user animation preferences and macOS accessibility settings:**

```swift
// ✅ Good - Respects both app and system preferences
@AppStorage("enableAnimations") private var enableAnimations = true
@AppStorage("reduceMotion") private var reduceMotion = false

VStack {
    // content
}
.animated(.smooth, value: someState)  // Automatically checks system settings

// Alternative using AnimationPreferences helper
let prefs = AnimationPreferences()
.animation(prefs.defaultAnimation, value: someState)
```

**Available animation preferences:**
- `enableAnimations`: Master toggle for all animations
- `reduceMotion`: Use linear animations instead of springs/easing
- `showLoadingIndicators`: Show progress indicators
- `compactMode`: Use denser layouts with reduced spacing
- `showEmptyStateIllustrations`: Show illustrations in empty states

**macOS Accessibility Integration:**
The app automatically respects these system settings:
- **Reduce Motion** (`NSWorkspace.shared.accessibilityDisplayShouldReduceMotion`)
  - Overrides app animation preferences when enabled

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AndyBold/container-manager](https://github.com/AndyBold/container-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
