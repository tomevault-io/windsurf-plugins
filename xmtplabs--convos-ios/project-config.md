---
trigger: always_on
description: This document contains project-specific conventions and best practices for the Convos iOS codebase.
---

# Convos iOS - Codebase Best Practices

This document contains project-specific conventions and best practices for the Convos iOS codebase.

> **Designers**: If you're a designer using Claude Code, see [.claude/DESIGNER.md](.claude/DESIGNER.md) for simplified instructions and quality gates.

## Architecture & Organization

### Project Structure
- **ConvosCore**: Swift Package containing app-specific business logic, models, services, repositories, writers, GRDB database, XMTP client, and the profile system (`Sources/ConvosCore/Profiles/` — AES-256-GCM image encryption, encrypted image loading, ProfileUpdate/ProfileSnapshot codecs)
- **ConvosAppData**: Shared foundation package — protobuf types (ConversationCustomMetadata, ConversationProfile, EncryptedImageRef), serialization (Base64URL, DEFLATE), profile helpers
- **ConvosInvites**: Reusable invite system package — cryptographic tokens, join request processing, invite tag storage (120 tests)
- **ConvosCoreiOS**: iOS-specific implementations needed by ConvosCore (e.g., `UIImage` handling, push notification registration)
- **Main App (Convos)**: Views and ViewModels only (SwiftUI with UIKit integration where needed)
- **App Clips**: Separate target for lightweight experiences
- **Notification Service**: Extension for push notification handling

### Module Architecture
- All business logic, models, and services go in `ConvosCore`
- iOS-specific code that ConvosCore needs goes in `ConvosCoreiOS`
- Views and ViewModels go in the main app target
- Use protocols for dependency injection (e.g., `SessionManagerProtocol`)

### ConvosCore Platform Independence

**ConvosCore must compile on macOS** to enable fast test execution without the iOS Simulator. This means:

- **Never import UIKit** in ConvosCore - use cross-platform alternatives
- **Never use `#if canImport(UIKit)`** conditionals - they break macOS compilation
- **Never use iOS-specific types** like `UIImage`, `UIColor`, `UIApplication`

**Cross-platform type aliases** are defined in `ConvosCore/Sources/ConvosCore/Utilities/`:
```swift
// ImageType.swift
#if canImport(AppKit)
public typealias ImageType = NSImage  // macOS
#else
public typealias ImageType = UIImage  // iOS
#endif
```

Use `ImageType` instead of `UIImage` throughout ConvosCore.

### ConvosCoreiOS Bridge Package

When ConvosCore needs iOS-specific functionality, use the **ConvosCoreiOS** package as a bridge:

1. **Define a protocol in ConvosCore** that describes what you need
2. **Implement the protocol in ConvosCoreiOS** using iOS-specific APIs
3. **Inject the implementation** from the main app at runtime

**Example - Push Notification Registration:**
```swift
// In ConvosCore - protocol definition
public protocol PushNotificationRegistering {
    func registerForRemoteNotifications() async throws -> Data
}

// In ConvosCoreiOS - iOS implementation
public final class PushNotificationManager: PushNotificationRegistering {
    public func registerForRemoteNotifications() async throws -> Data {
        // Uses UIApplication.shared.registerForRemoteNotifications()
    }
}

// In main app - injection
let service = MyService(pushManager: PushNotificationManager())
```

**What goes where:**
| Location | Content |
|----------|---------|
| ConvosCore | Protocols, business logic, cross-platform types |
| ConvosCoreiOS | UIKit implementations, iOS system integrations |
| Main App | SwiftUI views, dependency injection, app lifecycle |

### XMTP SDK Abstraction Pattern
To enable testability and avoid tight coupling to the XMTP iOS SDK, Convos uses protocol wrappers around XMTP types:

- **`XMTPClientProvider`**: Protocol that mirrors the XMTP SDK's client interface
- Allows dependency injection of mock XMTP clients in tests
- Prevents direct usage of XMTP SDK types throughout the codebase
- Enables testing without requiring live XMTP network connections

**Pattern:**
```swift
// ✅ Good - Use the protocol wrapper
func myFunction(client: any XMTPClientProvider) { }

// ❌ Bad - Direct XMTP SDK usage
func myFunction(client: XMTPiOS.Client) { }
```

This pattern applies to other XMTP types as well - prefer protocol wrappers for dependency injection and testability.

## SwiftUI Conventions

### State Management
- **Modern Observation Framework**: Use `@Observable` with `@State` for new code
  ```swift
  @Observable
  class MyViewModel {
      var property: String = ""
  }

  // In views:
  @State private var viewModel = MyViewModel()
  ```
- Legacy code may still use `ObservableObject` with `@StateObject`/@ObservedObject`

### Button Pattern
Always extract button actions to avoid closure compilation errors:
```swift
// ✅ Good
let action = { /* action code */ }
Button(action: action) {
    // view content
}

// ❌ Bad - causes compilation issues
Button(action: { /* action */ }) {
    // view content
}
```

### Preview Support
Use `@Previewable` for preview state variables:
```swift
@Previewable @State var text: String = "Preview"
```

### View Modes for Multi-Entry-Point Surfaces

When the same screen is shown from multiple entry points with subtly
different behavior (e.g., a card with extra actions in some contexts), prefer
parameterizing the view with a mode enum over duplicating the view per
surface:

```swift

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xmtplabs/convos-ios](https://github.com/xmtplabs/convos-ios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
