---
trigger: always_on
description: The DuckDuckGo browser apps for iOS and macOS leverage **Apple's Unified Logging System** for capturing telemetry and debugging information. This system enables efficient tracking of app behavior, issue diagnosis, and performance monitoring in a structured and privacy-conscious manner.
---


# Logging Guidelines & Telemetry Capture

## Overview

The DuckDuckGo browser apps for iOS and macOS leverage **Apple's Unified Logging System** for capturing telemetry and debugging information. This system enables efficient tracking of app behavior, issue diagnosis, and performance monitoring in a structured and privacy-conscious manner.

**Key Benefits**:
- **Privacy-first**: Built-in privacy controls for sensitive data
- **Performance**: Optimized for minimal overhead
- **Integration**: Native Apple ecosystem support
- **Debugging**: Rich contextual information and filtering

## How to Log

### Using the Logger Class

We utilize the `Logger` class from Apple's `os` framework for all logging activities:

```swift
import os

// Basic logging examples
Logger.yourFeatureName.debug("Something to log, with info: \(infoVar)")
Logger.anotherFeatureName.error("Some error happened: \(error.localizedDescription, privacy: .public)")
Logger.networking.info("API request completed for endpoint: \(endpoint, privacy: .public)")
Logger.performance.debug("Operation took \(duration)ms to complete")
```

### Creating Custom Loggers

#### Single Feature Logger

For new features, create a dedicated logger file named `Logger+YourFeatureName.swift`:

```swift
import os

public extension Logger {
    static var yourFeatureName: Logger = { 
        Logger(subsystem: "Your Feature Name", category: "") 
    }()
    
    static var anotherFeatureName: Logger = { 
        Logger(subsystem: "Another feature name", category: "Subsystem in the feature") 
    }()
}
```

#### Multiple Feature Loggers

For related features, add to existing logger extensions (e.g., `Logger+Multiple.swift`):

```swift
import os

public extension Logger {
    // Networking loggers
    static var networking: Logger = { 
        Logger(subsystem: "Networking", category: "API") 
    }()
    
    static var cache: Logger = { 
        Logger(subsystem: "Networking", category: "Cache") 
    }()
    
    // UI loggers  
    static var tabManagement: Logger = { 
        Logger(subsystem: "UI", category: "Tab Management") 
    }()
    
    static var bookmarks: Logger = { 
        Logger(subsystem: "UI", category: "Bookmarks") 
    }()
}
```

### Logger Placement Strategy

**Framework/Package Level**: For shared functionality across iOS and macOS
```swift
// In BrowserServicesKit
public extension Logger {
    static var secureVault: Logger = { 
        Logger(subsystem: "BrowserServicesKit", category: "SecureVault") 
    }()
    
    static var sync: Logger = { 
        Logger(subsystem: "BrowserServicesKit", category: "Sync") 
    }()
}
```

**App Level**: For platform-specific features
```swift
// In iOS app
public extension Logger {
    static var widgets: Logger = { 
        Logger(subsystem: "iOS App", category: "Widgets") 
    }()
}

// In macOS app  
public extension Logger {
    static var windowManagement: Logger = { 
        Logger(subsystem: "macOS App", category: "Window Management") 
    }()
}
```

## Subsystem and Category Guidelines

### Subsystem Naming

**Purpose**: Corresponds to large functional areas of your app

**Examples**:
- `"Networking"` - All network-related functionality
- `"UI"` - User interface components
- `"Data Storage"` - Database and persistence
- `"Security"` - Authentication and encryption
- `"Performance"` - Performance monitoring and optimization

### Category Naming

**Purpose**: Specific components or features within subsystems

**Examples**:
```swift
// Networking subsystem categories
Logger(subsystem: "Networking", category: "API Calls")
Logger(subsystem: "Networking", category: "Cache Management")
Logger(subsystem: "Networking", category: "Request Retry")

// UI subsystem categories  
Logger(subsystem: "UI", category: "Tab Management")
Logger(subsystem: "UI", category: "Settings")
Logger(subsystem: "UI", category: "Bookmarks")

// Data Storage subsystem categories
Logger(subsystem: "Data Storage", category: "SecureVault")
Logger(subsystem: "Data Storage", category: "Core Data")
Logger(subsystem: "Data Storage", category: "User Defaults")
```

## Log Levels and Privacy

### Choosing Log Levels

#### `debug` - Development and Troubleshooting
- **Purpose**: Verbose output for development debugging
- **Retention**: Short-lived in memory
- **Use cases**: Variable values, execution flow, temporary debugging

```swift
Logger.networking.debug("Request headers: \(headers)")
Logger.ui.debug("User tapped button at coordinates: \(point)")
Logger.performance.debug("Cache hit for key: \(key)")
```

#### `info` - Important Events
- **Purpose**: Interesting or important information
- **Retention**: Longer than debug, available for analysis
- **Use cases**: User actions, system state changes, feature usage

```swift
Logger.auth.info("User successfully authenticated")
Logger.sync.info("Sync operation completed with \(itemCount) items")
Logger.features.info("Feature flag \(flagName, privacy: .public) enabled")
```

#### `error` - Handled Errors
- **Purpose**: Something went wrong but was handled gracefully

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
