---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OpenCoreAnimation targets full CoreAnimation (QuartzCore) API compatibility for WebAssembly (WASM) environments. Current completion is measured per API and renderer path.

### Core Principle: Full Compatibility

**The target API must be 100% compatible with CoreAnimation.** This means:
- Identical type names, method signatures, and property names
- Implemented behavior and semantics must be independently validated against CoreAnimation
- Compatibility claims must identify the exercised surface and evidence
- **Property types must exactly match Apple's Swift interface** — Do NOT guess types based on property names or similar APIs. Always verify against Apple's documentation. A type mismatch (`Float` vs `CGFloat`) is a compile error for users.
- **Foundation types unavailable on WASM** (`NSNumber`, `NSValue`, etc.) must be replaced with the equivalent Swift numeric type that preserves the same precision and semantics (e.g., `[NSNumber]?` → `[CGFloat]?`)

### Float vs CGFloat の使い分け

- **公開API**: Appleの定義に従う。自分で判断しない。
- **内部実装**: `CGFloat`プロパティから流れる値の計算には`CGFloat`を使う。`Float`はGPUに渡すデータ（頂点、uniform）、および`Float`で定義された既存APIとの境界でのみ使う。

### How `canImport` Works

Users of this library will write code like:

```swift
#if canImport(QuartzCore)
import QuartzCore
#else
import OpenCoreAnimation
#endif

let layer = CALayer()
layer.frame = CGRect(x: 0, y: 0, width: 100, height: 100)
```

- **When QuartzCore is available** (iOS, macOS, etc.): Users import QuartzCore directly
- **When QuartzCore is NOT available** (WASM): Users import OpenCoreAnimation

## Build Commands

```bash
swift build
perl -e 'alarm 300; exec @ARGV' -- \
  xcodebuild test -scheme OpenCoreAnimation -destination 'platform=macOS' \
  SWIFT_COMPILATION_MODE=wholemodule SWIFT_ENABLE_BATCH_MODE=NO \
  -only-testing:OpenCoreAnimationTests
TOOLCHAINS=org.swift.64202607171a xcrun swift build \
  --swift-sdk swift-6.4.x-DEVELOPMENT-SNAPSHOT-2026-07-17-a_wasm
cd Tests/e2e && npm test
```

## Platform Strategy

**OpenCoreAnimation primarily targets WASM/Web environments**, but includes native implementations for testing purposes.

### Production Use

- **WASM**: Uses WebGPU via swift-webgpu, JavaScriptKit for browser APIs
- **Native platforms (iOS, macOS)**: Users should import Apple's QuartzCore directly

### Native Test Support

To enable focused native tests on macOS/Linux, the library includes fallback implementations:

| Component | WASM Implementation | Native (Test) Implementation |
|-----------|---------------------|------------------------------|
| Timing | `performance.now()` | `ProcessInfo.systemUptime` |
| Display Link | `requestAnimationFrame` | `Timer` |
| Transactions | `setTimeout` | owning thread's `RunLoop.perform` |
| Renderer | `CAWebGPURenderer` | `CAMetalRenderer` |
| Graphics | `OpenCoreGraphics` | `CoreGraphics` (re-exported) |

These native implementations are **for testing only** and should not be used in production.

### Conditional Compilation

Use `#if arch(wasm32)` to distinguish between WASM and native platforms:

```swift
#if arch(wasm32)
import JavaScriptKit
// WASM-specific implementation
#else
import Foundation
// Native fallback for testing
#endif
```

For graphics types, `OpenCoreAnimation.swift` re-exports the appropriate module:

```swift
#if canImport(CoreGraphics)
@_exported import CoreGraphics      // Native: use Apple's CoreGraphics
#else
@_exported import OpenCoreGraphics  // WASM: use OpenCoreGraphics
#endif
```

### WASM-Specific Features

For WASM-specific features like timing and display links, use JavaScriptKit directly:

```swift
import OpenCoreGraphics
import JavaScriptKit

// Use JavaScript APIs for timing
let performance = JSObject.global.performance
let timestamp = performance.now().number ?? 0

// Use requestAnimationFrame for display sync
let callback = JSClosure { ... }
_ = JSObject.global.requestAnimationFrame!(callback)
```

## Testing

Uses Swift Testing framework (not XCTest):

```swift
import Testing
import OpenCoreGraphics
@testable import OpenCoreAnimation

@Test func testCALayerFrame() {
    let layer = CALayer()
    layer.frame = CGRect(x: 10, y: 20, width: 100, height: 200)
    #expect(layer.frame == CGRect(x: 10, y: 20, width: 100, height: 200))
}
```

## WebGPU Rendering Backend

### Overview

OpenCoreAnimation **primarily targets WASM/Web environments** and uses **WebGPU** as its GPU rendering backend via [swift-webgpu](https://github.com/1amageek/swift-webgpu). This provides hardware-accelerated layer rendering comparable to Metal on Apple platforms.

**Key point**: On native platforms (iOS, macOS), users should import Apple's QuartzCore directly for production use. OpenCoreAnimation includes native fallback implementations (Metal renderer, Foundation-based timing) for **testing purposes only**.

### Dependency: swift-webgpu

```swift
// Package.swift dependency
.package(url: "https://github.com/1amageek/swift-webgpu.git", branch: "main")

// Target dependency
.target(
    name: "OpenCoreAnimation",
    dependencies: [
        "OpenCoreGraphics",
        .product(name: "WebGPU", package: "swift-webgpu")
    ]
)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [1amageek/OpenCoreAnimation](https://github.com/1amageek/OpenCoreAnimation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
