---
trigger: always_on
description: │   ├── Package.swift
---


# Shared Packages Development Guidelines

## Package Structure

### Standard Package Layout
```
SharedPackages/
├── FeatureName/
│   ├── Package.swift
│   ├── README.md
│   ├── Sources/
│   │   └── FeatureName/
│   │       ├── Public/          # Public API
│   │       ├── Internal/        # Internal implementation
│   │       └── Resources/       # Assets and resources
│   └── Tests/
│       └── FeatureNameTests/
│           └── FeatureTests.swift
```

### Package.swift Configuration
```swift
// swift-tools-version: 5.7
import PackageDescription

let package = Package(
    name: "FeatureName",
    platforms: [
        .iOS(.v15),
        .macOS(.v12)
    ],
    products: [
        .library(
            name: "FeatureName",
            targets: ["FeatureName"]
        )
    ],
    dependencies: [
        // Only include truly necessary dependencies
        .package(url: "https://github.com/DuckDuckGo/BrowserServicesKit", from: "1.0.0")
    ],
    targets: [
        .target(
            name: "FeatureName",
            dependencies: ["BrowserServicesKit"],
            resources: [
                .process("Resources")
            ]
        ),
        .testTarget(
            name: "FeatureNameTests",
            dependencies: ["FeatureName"]
        )
    ]
)
```

## Cross-Platform Compatibility

### Platform-Specific Code
```swift
#if os(iOS)
import UIKit
public typealias PlatformView = UIView
public typealias PlatformViewController = UIViewController
public typealias PlatformColor = UIColor
#elseif os(macOS)
import AppKit
public typealias PlatformView = NSView
public typealias PlatformViewController = NSViewController
public typealias PlatformColor = NSColor
#endif

// Use platform-agnostic types
public protocol CrossPlatformViewProtocol {
    var backgroundColor: PlatformColor? { get set }
}
```

### Conditional Compilation
```swift
public class FeatureManager {
    public func performAction() {
        #if os(iOS)
        performIOSAction()
        #elseif os(macOS)
        performMacOSAction()
        #endif
    }
    
    #if os(iOS)
    private func performIOSAction() {
        // iOS-specific implementation
    }
    #endif
    
    #if os(macOS)
    private func performMacOSAction() {
        // macOS-specific implementation
    }
    #endif
}
```

## API Design

### Public API Guidelines
```swift
// Mark public APIs clearly
public protocol FeatureServiceProtocol {
    func fetchData() async throws -> [Item]
}

public final class FeatureService: FeatureServiceProtocol {
    // Use dependency injection
    private let networkClient: NetworkClientProtocol
    
    public init(networkClient: NetworkClientProtocol) {
        self.networkClient = networkClient
    }
    
    public func fetchData() async throws -> [Item] {
        // Implementation
    }
}
```

### Internal Implementation
```swift
// Keep implementation details internal
internal final class FeatureImplementation {
    // Not exposed to package consumers
}

// Use extensions for internal helpers
internal extension String {
    var sanitized: String {
        // Internal helper method
    }
}
```

## Resource Management

### Bundled Resources
```swift
public enum FeatureResources {
    private static let bundle = Bundle.module
    
    public static var configuration: Data {
        guard let url = bundle.url(forResource: "config", withExtension: "json"),
              let data = try? Data(contentsOf: url) else {
            fatalError("Missing required resource: config.json")
        }
        return data
    }
    
    public static func image(named name: String) -> PlatformImage? {
        #if os(iOS)
        return UIImage(named: name, in: bundle, with: nil)
        #elseif os(macOS)
        return bundle.image(forResource: name)
        #endif
    }
}
```

## Dependency Management

### Minimal Dependencies
```swift
// Avoid unnecessary dependencies
// Bad: Importing entire framework for one function
import HeavyFramework

// Good: Implement minimal version or use protocol
protocol DateFormatterProtocol {
    func string(from date: Date) -> String
}
```

### Version Management
```swift
// Use semantic versioning
// Package.swift
dependencies: [
    .package(url: "https://github.com/example/package", 
             from: "1.0.0"),  // Allows 1.x.x
    .package(url: "https://github.com/example/strict", 
             exact: "2.1.0"), // Exact version
    .package(url: "https://github.com/example/range", 
             "1.0.0"..<"2.0.0") // Version range
]
```

## Testing Shared Packages

### Cross-Platform Tests
```swift
import XCTest
@testable import FeatureName

final class FeatureTests: XCTestCase {
    func testCrossPlatformBehavior() {
        let feature = Feature()
        
        #if os(iOS)
        XCTAssertNotNil(feature.iosSpecificProperty)
        #elseif os(macOS)
        XCTAssertNotNil(feature.macOSSpecificProperty)
        #endif
        
        // Test common behavior
        XCTAssertEqual(feature.commonProperty, expectedValue)
    }
}
```

### Test Utilities
```swift
// Provide test utilities in a separate target
public extension XCTestCase {
    func waitForCondition(
        _ condition: @autoclosure () -> Bool,
        timeout: TimeInterval = 1.0,
        message: String = "Condition not met"
    ) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
