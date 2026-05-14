---
trigger: always_on
description: ALWAYS check this manual first before acting.
---

# AGENT INSTRUCTIONS

ALWAYS check this manual first before acting.

This repo is a modern Swift CLI alternative to Fastlane. Your task is to improve the repo when the user asks you to. Always start your feature implementation in TDD manner, i.e. start with a unit test. Ensure the system is fully intact on every step, i.e. run unit tests frequently.

## Modern Swift 6 CLI Development Guide

This document outlines the standard for high-performance, race-safe Swift command-line tools. It focuses on **Strict Concurrency** and **Structured Concurrency** while eliminating legacy patterns.

---

## 1. Best Practices: Modern Usage

### A. The Async Entry Point
In 2026, we avoid `main.swift` top-level code in favor of a structured `@main` entry point using `AsyncParsableCommand`.

```swift
import Foundation
import ArgumentParser // Standard CLI library

@main
struct ModernTool: AsyncParsableCommand {
    @Option(name: .shortAndLong, help: "Number of threads to simulate")
    var count: Int = 5

    // GOOD: Native async entry point ensures the tool waits for all tasks
    func run() async throws {
        print("🚀 Starting modern Swift 6 process...")

        let tracker = ProgressTracker()

        // GOOD: Structured Concurrency (TaskGroup)
        // Automatically manages child task lifetimes
        try await withThrowingTaskGroup(of: Void.self) { group in
            for i in 1...count {
                group.addTask {
                    try await performWork(id: i, tracker: tracker)
                }
            }
        }

        let finalCount = await tracker.total
        print("✅ Finished. Total items processed: \(finalCount)")
    }
}
```

Use code with caution.

B. Thread-Safe State (Actors)
Use actors to manage shared mutable state. This replaces the manual DispatchQueue locks of previous years.
```swift
// GOOD: Protects shared state with compile-time safety
actor ProgressTracker {
    private(set) var total = 0

    func increment() {
        total += 1
    }
}
```

C. Cooperative Cancellation

CLI tools must respond to user interrupts (Ctrl+C). Modern Swift handles this via Task.checkCancellation().
swift
func performWork(id: Int, tracker: ProgressTracker) async throws {
    // GOOD: Periodically check if the user or system cancelled the task
    try Task.checkCancellation()

    // Simulate non-blocking work
    try await Task.sleep(for: .seconds(Double.random(in: 0.5...1.5)))

    await tracker.increment()
    print("  [Task \(id)] Work complete.")
}
Use code with caution.

2. Anti-Patterns: Codes to Avoid
❌ Anti-Pattern: Unstructured "Fire-and-Forget" Tasks
In a CLI, the program terminates as soon as run() returns. Using Task { } allows work to start, but the program will likely exit before it finishes.
```swift
// BAD: This work will be killed prematurely
func run() {
    Task {
        await someLongProcess() // Tool exits before this completes
    }
}
```

❌ Anti-Pattern: Blocking the Concurrent Thread Pool
Calling synchronous, blocking functions inside an async function prevents the Swift runtime from reusing that thread for other tasks.

```swift
// BAD: Blocks a thread in the concurrency pool
func processData() async {
    let data = try! Data(contentsOf: someLargeFileURL) // Synchronous/Blocking
}

// GOOD: Use async-native file I/O or URLSession
```

❌ Anti-Pattern: Force Unwrapping and Manual Casting
Modern Swift emphasizes safety. Using ! leads to fragile CLI tools that provide no helpful error messages before crashing.
```swift
// BAD: Crash without explanation if the URL is malformed
let url = URL(string: userInput)!

// GOOD: Throw a descriptive error
guard let url = URL(string: userInput) else {
    throw ValidationError("Invalid URL provided.")
}
```

❌ Anti-Pattern: Silencing Errors
Using try? in a CLI is problematic because the user loses all diagnostic information about why a command failed.
```swift
// BAD: User has no idea why the file failed to save
try? data.write(to: path)

// GOOD: Propagate error so it can be printed to stderr
try data.write(to: path)
```

❌ Anti-Pattern: Editing Generated Code or OpenAPI Specs
NEVER edit:
- Files in `Generated/` directories - auto-generated and will be overwritten
- OpenAPI spec files (`openapi.json`) - these are 3rd party specs from Apple that must remain unchanged

```swift
// BAD: Editing Sources/API/*/Generated/Types.swift
// BAD: Editing Sources/API/*/openapi.json

// GOOD: Handle at domain layer (TestflightAPI wrapper for example)
```

❌ Anti-Pattern: Excessive verbose comments

```swift
// BAD: excessive comments

// Should have created a certificate
XCTAssertEqual(mockAPI.certificates.count, 1)
let cert = mockAPI.certificates.first
XCTAssertEqual(cert?.type, .development)

// Should have stored certificate in git (cer + p12)
// Cert ID is generated, so we need to use the one from API
guard let createdCert = cert else { return }
let certPath = "certs/ios/DEVELOPMENT/\(createdCert.id).cer"
let p12Path = "certs/ios/DEVELOPMENT/\(createdCert.id).p12"

// GOOD: Be concise and don't pour water with your comments

XCTAssertEqual(mockAPI.certificates.count, 1)
let cert = mockAPI.certificates.first
XCTAssertEqual(cert?.type, .development)

guard let cert else { return }

let certPath = "certs/ios/DEVELOPMENT/\(createdCert.id).cer"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [platacard/blimp](https://github.com/platacard/blimp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
