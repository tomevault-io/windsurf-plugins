---
trigger: always_on
description: This document provides Oak-specific patterns and examples.
---

# Oak Framework - AI Coding Assistant Instructions

This document provides Oak-specific patterns and examples.

> Repository policy: For high-level, tool-agnostic guidance shared across assistants, see `AI_GUIDELINES.md`. Treat that file as the canonical repo policy; Also refer to [BasicSystemPrompt](BasicSystemPrompt.md) for platform rules, quality and best practices.

## Purpose & Scope (For Agents)
- Optimize for concise, correct changes; avoid large refactors unless requested.
- Follow Oak’s FSM model: pure `update`, effects in `Effect`, Swift concurrency.
- Respect terminal states and `Sendable` boundaries; do not add global state.
- Keep edits surgical and in-repo; don’t invent APIs or move files unnecessarily.

## Quick Rules For Agents
- Safety: No side effects in `update`. Effects may send events; action effects process synchronously.
- Concurrency: Honor isolation; use `@Sendable` where values cross boundaries.
- State: Prefer enums; implement `isTerminal` precisely. Don’t ignore terminal transitions.
- Testing: Use existing XCTest targets. Avoid real timers/networking in tests.
- Docs: Update docs when changing effect/terminal behavior. Link to `AI_GUIDELINES.md`.

## Workflows At A Glance
- Run tests: `swift test` (targets: `OakTests`, `OakBenchmarks`).
- Format: `./Scripts/formatCode.sh`.
- Docs: `./Scripts/previewDocs.sh` or `./Scripts/generateDocs.sh`.
- SwiftUI usage: Prefer `TransducerView` or `ObservableTransducer` over `ObservableObject`.

## Project Overview

Oak is a Swift finite state machine (FSM) library built on structured concurrency, designed for type-safe, reactive application architecture with SwiftUI integration. It emphasizes pure functional state transitions separated from side effects.

## Core Architecture Concepts

### Transducer Definition Pattern
```swift
// Simple Transducer - no effects, direct output
enum SimpleCounter: Transducer {
    // REQUIRED: State definition
    enum State: NonTerminal {
        case start
        case idle(count: Int)
    }
    
    // REQUIRED: Event definition
    enum Event {
        case start
        case increment, decrement
    }
    
    // REQUIRED: Output typealias when producing output
    typealias Output = Int
    
    // REQUIRED: update function returning output directly
    static func update(_ state: inout State, event: Event) -> Output {
        switch (state, event) {
        case (.start, .start):
            state = .idle(count: 0)
            return 0
        case (.idle(let count), .increment):
            state = .idle(count: count + 1)
            return count + 1
        case (.idle(let count), .decrement):
            state = .idle(count: max(0, count - 1))
            return max(0, count - 1)
        default:
            // When reaching in the `default` case , it may indicate 
            // a violation of the assumptions about the system. You 
            // may want to log or handle this case.

            // Handle unexpected combinations
            return 0
        }
    }
    
    // RECOMMENDED: Initial state definition
    static var initialState: State { .start }
    
    // REQUIRED for Moore automata: Initial output function
    static func initialOutput(initialState: State) -> Output? {
        switch initialState {
        case .start: return nil
        case .idle(let count): return count
        }
    }
}

// EffectTransducer - with async effects
enum MyTransducer: EffectTransducer {
    // REQUIRED: State definition
    enum State: Terminable {
        case start
        case idle, processing, finished
        var isTerminal: Bool { self == .finished }
    }
    
    // REQUIRED: Event definition  
    enum Event {
        case start
        case process, complete
    }
    
    // REQUIRED: Output typealias when producing output
    typealias Output = String
    
    // REQUIRED: update function returning effects
    static func update(_ state: inout State, event: Event) -> Effect? {
        switch (state, event) {
        case (.start, .start):
            state = .idle
            return nil // Ready to handle events
        case (.idle, .process):
            state = .processing
            return someAsyncEffect()
        // ... other transitions
        default:
            return nil
        }
    }
    
    // RECOMMENDED: Initial state definition
    static var initialState: State { .start }
    
    // REQUIRED for Moore automata: Initial output function
    // Only handles valid initial states (.start in this case)
    static func initialOutput(initialState: State) -> Output? {
        switch initialState {
        case .start: return nil
        // Only include cases for states that can be initial states
        }
    }
}
```

### Transducer Types
- **`Transducer`**: Basic FSM with pure state transitions, returns `Output` directly
- **`EffectTransducer`**: Advanced FSM that can trigger side effects, returns `Effect?` or `(Effect?, Output)`
- **`BaseTransducer`**: Type container for composition without implementation requirements

### State Management Philosophy
```swift
// Prefer SUM types (enums) over PRODUCT types (structs) for State
enum State: Terminable {
    case idle(value: Int)
    case loading
    case finished(result: String)
    
    var isTerminal: Bool { 

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [couchdeveloper/Oak](https://github.com/couchdeveloper/Oak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
