---
trigger: always_on
description: The Configurations module provides a declarative API for setting up Snowplow trackers. It uses the builder pattern with protocol-based configuration objects that can be composed, serialized, and applied at runtime or through remote configuration. Each configuration type controls a specific aspect of tracker behavior.
---

# Configurations Module - CLAUDE.md

## Module Overview

The Configurations module provides a declarative API for setting up Snowplow trackers. It uses the builder pattern with protocol-based configuration objects that can be composed, serialized, and applied at runtime or through remote configuration. Each configuration type controls a specific aspect of tracker behavior.

**Key Patterns**: Builder pattern, Protocol composition, Configuration merging, Serialization
**Core Types**: TrackerConfiguration, NetworkConfiguration, EmitterConfiguration, SessionConfiguration

## Configuration Hierarchy

```
ConfigurationProtocol (Base)
├── TrackerConfiguration     # Core tracker settings
├── NetworkConfiguration     # Collector endpoint
├── EmitterConfiguration     # Batching & sending
├── SessionConfiguration     # Session management
├── SubjectConfiguration     # User properties
├── GDPRConfiguration       # Privacy settings
└── GlobalContextsConfiguration # Context generators
```

## Configuration Patterns

### Builder Pattern Usage
```swift
// ✅ Fluent configuration
let tracker = Snowplow.createTracker(
    namespace: "app",
    endpoint: "https://collector.com"
) {
    TrackerConfiguration()
        .appId("my-app")
        .base64Encoding(false)
    SessionConfiguration(
        foregroundTimeout: 300,
        backgroundTimeout: 150
    )
}

// ❌ Imperative configuration
let tracker = createTracker()
tracker.appId = "my-app"
tracker.base64 = false
```

### Configuration Composition
```swift
// ✅ Multiple configurations
let configs: [ConfigurationProtocol] = [
    TrackerConfiguration(),
    NetworkConfiguration(endpoint: url),
    EmitterConfiguration(),
    SessionConfiguration()
]

// ❌ Mixed configuration types
let config = TrackerConfiguration()
config.networkEndpoint = url  // Wrong layer
```

## TrackerConfiguration

### Core Properties
```swift
// ✅ Essential tracker settings
TrackerConfiguration()
    .appId("com.example.app")
    .devicePlatform(.mobile)
    .base64Encoding(true)
    .logLevel(.debug)

// ❌ Missing required settings
TrackerConfiguration()  // Defaults may not be appropriate
```

### Context Control
```swift
// ✅ Explicit context enabling
TrackerConfiguration()
    .sessionContext(true)
    .platformContext(true)
    .applicationContext(false)  // Opt-out

// ❌ Assume all contexts enabled
TrackerConfiguration()
    .trackAllContexts(true)  // Too broad
```

### Platform Context Properties
```swift
// ✅ Select specific properties
TrackerConfiguration()
    .platformContextProperties([
        .osType, .osVersion, .deviceModel
    ])

// ❌ Track everything
.platformContextProperties(nil)  // May include PII
```

## NetworkConfiguration

### Endpoint Setup
```swift
// ✅ Secure endpoint with method
NetworkConfiguration(endpoint: "https://collector.com")
    .method(.post)
    .customPostPath("/com.snowplow/tp2")

// ❌ Insecure configuration
NetworkConfiguration(endpoint: "http://collector.com")
    .method(.get)  // GET has size limits
```

### Custom Headers
```swift
// ✅ Add authentication headers
NetworkConfiguration(endpoint: url)
    .requestHeaders(["Authorization": "Bearer token"])

// ❌ Headers in wrong config
TrackerConfiguration()
    .headers(["Auth": "token"])  // Not supported
```

## EmitterConfiguration

### Buffer Management
```swift
// ✅ Configure batching
EmitterConfiguration()
    .bufferOption(.single)  // Real-time
    // or
    .bufferOption(.defaultGroup)  // Batch of 25

// ❌ Arbitrary buffer sizes
.bufferOption(.custom(1000))  // Too large
```

### Request Callbacks
```swift
// ✅ Handle emit results
EmitterConfiguration()
    .requestCallback { count, success in
        if !success {
            logError("Failed to send \(count) events")
        }
    }

// ❌ Ignore failures
EmitterConfiguration()  // Silent failures
```

### Thread Pool Control
```swift
// ✅ Appropriate thread count
EmitterConfiguration()
    .threadPoolSize(2)  // Mobile-friendly

// ❌ Too many threads
.threadPoolSize(10)  // Battery drain
```

## SessionConfiguration

### Timeout Settings
```swift
// ✅ Reasonable timeouts
SessionConfiguration(
    foregroundTimeout: Measurement(value: 30, unit: .minutes),
    backgroundTimeout: Measurement(value: 5, unit: .minutes)
)

// ❌ Too short timeouts
SessionConfiguration(
    foregroundTimeout: Measurement(value: 10, unit: .seconds)
)  // Too many sessions
```

### Session Callbacks
```swift
// ✅ Track session lifecycle
SessionConfiguration(
    onSessionStateUpdate: { state in
        analytics.track("session_update", state)
    }
)

// ❌ Heavy operations in callback
onSessionStateUpdate: { state in
    saveToDatabase(state)  // Blocks tracking
}
```

## SubjectConfiguration

### User Properties
```swift
// ✅ Set user information
SubjectConfiguration()
    .userId("user-123")
    .networkUserId(UUID())
    .domainUserId("domain-user")

// ❌ PII in wrong fields
SubjectConfiguration()
    .userId("john@example.com")  // Use hashed ID
```

### Screen Dimensions
```swift

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [snowplow/snowplow-ios-tracker](https://github.com/snowplow/snowplow-ios-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
