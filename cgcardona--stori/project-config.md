---
trigger: always_on
description: You are working on TellUrStoriDAW, an innovative digital audio workstation that combines:
---

# TellUrStoriDAW - Digital Audio Workstation with NFT Tokenization
# Swift/SwiftUI DAW + Avalanche L1 Blockchain

## Project Overview
You are working on TellUrStoriDAW, an innovative digital audio workstation that combines:
- Professional DAW functionality in Swift/SwiftUI
- NFT tokenization of STEMs on custom Avalanche L1
- Comprehensive marketplace for trading music NFTs

## Core Technologies
- **App**: Swift 6, SwiftUI, AVFoundation, Core Audio, Metal
- **Target**: macOS 14+ (Sonoma) - Required for @Observable macro
- **Blockchain**: Solidity, Hardhat, OpenZeppelin, Avalanche L1
- **Infrastructure**: IPFS, GraphQL (for indexer/marketplace when used)

## Development Philosophy
- **Performance First**: Audio applications demand < 10ms latency
- **Apple Ecosystem Excellence**: Leverage native frameworks and design patterns
- **Startup Velocity**: Balance rapid iteration with code quality
- **User Experience**: Prioritize intuitive, delightful interactions
- **Security**: Blockchain and audio data require robust security practices

## Swift/SwiftUI Guidelines

### Code Style
- Use Swift 6 modern concurrency (async/await, actors, structured concurrency)
- Prefer SwiftUI over UIKit for new development
- Follow Apple's Swift API Design Guidelines
- Use meaningful, descriptive names for all identifiers
- Leverage Swift's type system for safety and clarity

### Architecture Patterns
- **MVVM with @Observable**: Use the `@Observable` macro (NOT ObservableObject) for all view models and services
  - Only views reading specific properties re-render when those properties change (fine-grained updates)
  - Use `@ObservationIgnored` for internal state that shouldn't trigger UI updates
  - NEVER use `ObservableObject` or `@Published` - these cause broadcast invalidation and CPU spikes
- **Dependency Injection**: Use protocols and dependency containers
- **Modular Design**: Separate concerns into focused Swift packages
- **Reactive Programming**: Use Combine sparingly; prefer @Observable for UI state
- **Error Handling**: Use Result types and structured error handling
- **@preconcurrency import**: Place `@preconcurrency import ModuleName` as the **first import of that module** in the file (Swift compiler limitation). If the same module is imported earlier (e.g. `import struct Foundation.Data` before `@preconcurrency import Foundation`), the annotation is ignored and concurrency warnings appear. Applies to all modules (Foundation, Darwin, AVFoundation, third-party). Prefer migrating to @MainActor or @unchecked Sendable where appropriate; @preconcurrency only suppresses warnings and does not guarantee thread safety.

### Audio Development
- Always use real-time safe code in audio callbacks
- Avoid memory allocation in audio render threads
- Use SIMD and Accelerate framework for performance-critical operations
- Implement proper buffer management with circular buffers
- Profile audio code with Instruments regularly
- **@Observable for audio classes**: Use `@ObservationIgnored` for all audio nodes, timers, and internal state
  - Only UI-visible properties should be observable (e.g., levels, transport state)
  - Audio graph mutations, engine internals should NEVER trigger UI updates

### SwiftUI Best Practices (macOS 14+ with @Observable)
- Use `@State` for owning @Observable objects (replaces @StateObject)
- Use plain `var` for passed-in @Observable dependencies (replaces @ObservedObject)
- Use `@Bindable` when you need two-way bindings to @Observable properties
- Use `@Environment(TypeName.self)` for environment-injected @Observable objects
- Inject @Observable objects with `.environment(instance)` (NOT `.environmentObject()`)
- Use `@State` for simple view-local state (unchanged)
- Implement custom ViewModifiers for reusable styling
- Use PreferenceKeys for child-to-parent communication
- Optimize with LazyVStack/LazyHStack for large datasets
- **NEVER create #Preview blocks** - We don't use SwiftUI Previews in this project
- **NEVER use @StateObject, @ObservedObject, @EnvironmentObject** - These are legacy patterns

### Testing
- Write unit tests for all business logic
- Use XCTest for standard testing
- Implement UI tests for critical user flows
- Mock external dependencies (network, file system)
- Test audio components with synthetic data
- Aim for 90%+ code coverage

## Shell Scripting Guidelines

### Script Type Selection
- **New Scripts**: Use `zsh` (macOS default since Catalina)
- **Existing Scripts**: Keep as `bash` unless trivial to convert
- **Rationale**: TellUrStoriDAW is macOS-only, zsh is guaranteed available
- **Avoid Regressions**: Don't port bash→zsh unless conversion is obvious and safe

### Script Header
```zsh
#!/bin/zsh
# Script description here
set -e  # Exit on error
```

### When to Keep bash
- Script has bash-specific features (arrays, associative arrays)
- Complex logic that works in bash
- External scripts we don't own
- Risk of introducing bugs > benefit of zsh

### When to Use zsh
- All NEW shell scripts
- Simple utility scripts
- Scripts that leverage zsh features (better globbing, arrays)
- Scripts that need macOS-specific functionality

## Blockchain Development Guidelines

### Smart Contract Development

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cgcardona) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
