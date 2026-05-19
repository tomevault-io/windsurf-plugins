---
trigger: always_on
description: The DuckDuckGo browser has moved away from traditional AppDelegate-based lifecycle handling to a **state machine architecture**. While AppDelegate still exists, it has been significantly thinned out and now delegates responsibility to a structured state machine.
---


# App Lifecycle State Machine Architecture

## Overview

The DuckDuckGo browser has moved away from traditional AppDelegate-based lifecycle handling to a **state machine architecture**. While AppDelegate still exists, it has been significantly thinned out and now delegates responsibility to a structured state machine.

This approach ensures that lifecycle handling is **predictable, organized, and easy to maintain**.

## Architecture Components

### Three Core States

The architecture revolves around a state machine with three major states:

#### 1. **Launching** (Transient State)
- **Associated with**: `application(_:didFinishLaunchingWithOptions:)`
- **File**: `Launching.swift`
- **Purpose**: App's initial setup and dependency configuration
- **Responsibilities**:
  - Initialize all services and objects
  - Configure dependencies
  - Prepare UI components
  - Create `MainViewController` and set as `rootViewController`

#### 2. **Foreground** (Permanent State)
- **Associated with**: `applicationDidBecomeActive(_:)`
- **File**: `Foreground.swift`
- **Purpose**: App is fully interactive and user can engage with UI
- **Responsibilities**:
  - Resume suspended work
  - Handle user interactions
  - Manage active UI state

#### 3. **Background** (Permanent State)
- **Associated with**: `applicationDidEnterBackground(_:)`
- **File**: `Background.swift`
- **Purpose**: App is not active and UI is not visible
- **Responsibilities**:
  - Suspend ongoing work that doesn't need background execution
  - Prepare for potential termination
  - Handle background tasks

## State Machine Methods

### Core Transition Methods

All states implement specific methods for handling transitions:

#### `onTransition()`
- **When**: Called whenever the app enters that state from another state
- **Purpose**: Setup or cleanup during state transitions
- **Available in**: Foreground, Background

#### `willLeave()`
- **When**: Called before transitioning away from current state
- **Purpose**: Prepare for potential state change
- **Note**: Transition may be cancelled, in which case `didReturn()` is called
- **Available in**: Foreground, Background

#### `didReturn()`
- **When**: Called after successful transition to destination state OR when transition is cancelled
- **Purpose**: Finalize state entry or handle cancelled transition
- **Available in**: Foreground, Background

## Common Lifecycle Scenarios

### Cold App Start

```swift
// Flow: Launching → Foreground
1. Launching.init()                    // Initial setup
2. Foreground.onTransition()           // Enter foreground
3. Foreground.didReturn()              // Finalize foreground entry
```

### App Backgrounding

```swift
// Flow: Foreground → Background
1. Foreground.willLeave()              // Prepare to leave foreground
2. Background.onTransition()           // Enter background
3. Background.didReturn()              // Finalize background entry
```

### App Foregrounding

```swift
// Flow: Background → Foreground
1. Background.willLeave()              // Prepare to leave background
2. Foreground.onTransition()           // Enter foreground
3. Foreground.didReturn()              // Finalize foreground entry
```

### Interrupted Foreground (Alert/App Switcher)

```swift
// User receives alert but dismisses it
1. Foreground.willLeave()              // Attempt to leave
2. Foreground.didReturn()              // Cancelled - stay in foreground

// User opens App Switcher
1. Foreground.willLeave()              // Attempt to leave
// Two possible outcomes:
// A. User returns directly:
2. Foreground.didReturn()              // Return to foreground
// B. User switches to another app:
2. Background.onTransition()           // Actually transition to background
3. Background.didReturn()              // Finalize background entry
```

## Special iOS 18+ Scenarios

### Face ID Authentication on Cold Start

#### Successful Authentication
```swift
1. Launching.init()
2. Foreground.onTransition()
3. Foreground.didReturn()
```

#### Failed Authentication
```swift
1. Launching.init()
2. Background.onTransition()           // Goes to background on auth failure
3. Background.didReturn()
```

### DuckDuckGo Face ID Lock

#### Cold Start with DDG Face ID
```swift
1. Launching.init()
2. Foreground.onTransition()
3. Foreground.didReturn()
4. Foreground.willLeave()              // DDG auth triggers
5. Foreground.didReturn()              // User passes auth
```

### Critical Setup Failure

```swift
1. Launching.init() throws             // Setup fails (e.g., disk space)
2. Terminating.init()                  // App terminates
```

## Code Placement Patterns

### ⚙️ One-time Setup → `AppConfiguration`

**Location**: Inside `Launching.swift`

For setup that happens once and doesn't need ongoing lifecycle management:

```swift
class AppConfiguration {
    func start() {
        // Basic setup that doesn't require dependencies
        setupGlobalUserAgent()
        configureLogging()
    }
    
    func finalize() {
        // Setup that requires access to services or MainCoordinator

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
