---
trigger: always_on
description: **Project Type**: macOS Native Menu Bar Application
---

# KeyStats - AI Agent Development Guide

**Project Type**: macOS Native Menu Bar Application
**Language**: Swift 5.0
**Target**: macOS 13.0+ (Ventura)
**Architecture**: Event-driven singleton pattern with AppKit UI

## Quick Context

KeyStats is a privacy-focused macOS menu bar app that tracks keyboard/mouse statistics (counts only, no content logging). Core components:

- `InputMonitor`: Global event tap for keyboard/mouse monitoring
- `StatsManager`: Data aggregation and persistence (UserDefaults)
- `MenuBarController`: Status bar UI with compact dual-line display
- `StatsPopoverViewController`: Detailed statistics panel

**Privacy First**: NEVER log actual keystrokes, mouse positions, or user input content - only aggregate counts and distances.

---

## Decision Trees for Common Tasks

### 🔧 Before Making Any Code Changes

```
1. Read the relevant file(s) first
2. Check existing patterns and naming conventions
3. Verify Swift version compatibility (5.0+)
4. Consider thread safety (main vs background threads)
5. Check if changes affect permissions or privacy
```

### 🎯 When Adding New Features

```
New feature request?
├─ UI-related?
│  ├─ Menu bar display? → Update MenuBarController
│  └─ Detail panel? → Update StatsPopoverViewController
├─ Statistics tracking?
│  ├─ New metric? → Update StatsManager.Stats struct + persistence
│  └─ New event type? → Update InputMonitor event callbacks
├─ Data persistence? → Update StatsManager Codable conformance
└─ Permissions needed? → Update Info.plist + AppDelegate
```

### 🐛 When Debugging Issues

```
Issue type?
├─ No statistics updating?
│  ├─ Check: AXIsProcessTrusted() returns true
│  ├─ Check: InputMonitor.isMonitoring is true
│  └─ Check: Event tap is active (not nil)
├─ UI not updating?
│  ├─ Verify: Updates on DispatchQueue.main
│  └─ Check: menuBarUpdateHandler is set
├─ Data not persisting?
│  └─ Check: StatsManager.saveStats() called on changes
└─ Performance issues?
    └─ Review: Event sampling rates and debounce timers
```

---

## Critical Rules

### 🔴 MUST Follow (Security & Privacy)

- ✅ Only track counts and distances, NEVER content
- ✅ Always check accessibility permissions before monitoring
- ✅ Use `weak` references for delegates/closures to prevent leaks
- ✅ Dispatch UI updates on `DispatchQueue.main`
- ✅ Clean up event taps in `stopMonitoring()` and `deinit`

### 🟡 SHOULD Follow (Quality)

- ✅ One class per file, filename matches class name
- ✅ Use `// MARK: -` for code organization
- ✅ Use `guard` for early returns and validation
- ✅ Use descriptive names, avoid magic numbers
- ✅ Localize user-facing strings with `NSLocalizedString()`
- ✅ Ensure UI colors adapt to dark mode (use dynamic colors + `resolvedCGColor`/`resolvedColor`)
- ✅ When adding a new page/window/popover, add matching analytics at the same time: a `pageview` for the page itself and `click` events for key entry/actions, reusing the shared helper and stable event/property names

### 🌗 Dark/Light Theme Switching Notes (Critical)

- ✅ `CALayer.backgroundColor` / `borderColor` use `CGColor` (a static snapshot) and do not automatically follow appearance changes
- ✅ Do not cache `NSColor.controlBackgroundColor.withAlphaComponent(...)` and reuse it across updates (especially when launched in dark mode), as it may lock in the old appearance
- ✅ For "dynamic system color + alpha", always resolve under the current `effectiveAppearance` using a helper, e.g. `resolvedCGColor(color, alpha:for:)`
- ✅ Re-assign layer colors on every theme change; do not rely on existing `CGColor` values to auto-update
- ✅ Prefer multi-source appearance refresh triggers: `AppearanceTrackingView`, `NSApp.effectiveAppearance`, `AppleInterfaceThemeChangedNotification`, and `NSApplication.didBecomeActiveNotification`
- ✅ When debugging theme issues, log `app/view/window` appearance plus final layer RGBA first to distinguish "trigger path issues" from "color resolution issues"

### 🟢 RECOMMENDED (Best Practices)

- ✅ Document public APIs with `///` comments
- ✅ Use `private` for internal implementation details
- ✅ Implement Codable for data structures needing persistence
- ✅ Batch UI updates to reduce main thread blocking

---

## Build & Development Commands

### Building

```bash
# Development (Xcode - recommended)
open KeyStats.xcodeproj
# Press ⌘R to build and run

# Command line (Debug)
xcodebuild -project KeyStats.xcodeproj -scheme KeyStats -configuration Debug build

# Command line (Release)
xcodebuild -project KeyStats.xcodeproj -scheme KeyStats -configuration Release build
```

### Distribution

```bash
# Create DMG for distribution
./scripts/build_dmg.sh
```

### Testing

```bash
# Currently no automated tests
# When adding: Use XCTest framework in separate Tests target
xcodebuild test -project KeyStats.xcodeproj -scheme KeyStats
```

---

## Code Patterns & Examples

### Singleton Pattern (Thread-Safe)

```swift
class StatsManager {
    static let shared = StatsManager()
    private init() {
        // Load from persistence
    }
}
```

### Permission Checking

```swift
// Check permission status
let trusted = AXIsProcessTrusted()

// Request permissions with prompt
let options = [kAXTrustedCheckOptionPrompt.takeUnretainedValue() as String: true]
AXIsProcessTrustedWithOptions(options as CFDictionary)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [debugtheworldbot/keyStats](https://github.com/debugtheworldbot/keyStats) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
