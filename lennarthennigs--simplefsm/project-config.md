---
trigger: always_on
description: This file provides specific guidance for AI assistants (especially Claude Code) when working with the SimpleFSM test suite.
---

# SimpleFSM Test Suite - AI Assistant Guidelines

This file provides specific guidance for AI assistants (especially Claude Code) when working with the SimpleFSM test suite.

## Test Suite Overview

The SimpleFSM test suite is organized into 7 comprehensive phases covering all aspects of the finite state machine library:

1. **StateManagement** - State creation, callbacks, properties, and lifecycle
2. **TransitionTests** - Event-driven transitions, guards, and validation  
3. **TimedTransitions** - Time-based automatic transitions and timing behavior
4. **GlobalTransitions** - Global transitions using both helper methods and traditional approaches
5. **ErrorHandling** - Error codes, parameter validation, and edge cases
6. **AdvancedFeatures** - FSM reset, DOT generation, state tracking, and handlers
7. **Integration** - Real-world scenarios like traffic lights and complex state machines

## Testing Framework & Environment

### Dependencies
- **AUnit** (v1.7.1+) - Arduino unit testing framework
- **EpoxyDuino** - For native testing without hardware (epoxy-esp8266, epoxy-esp32)
- **FSMTestHelper** - Helper class for accessing private FSM methods in tests

### Supported Test Environments
- `epoxy-esp8266` - ESP8266 emulation (recommended for fast testing)
- `epoxy-esp32` - ESP32 emulation 
- `Wemos_test` - Real ESP8266 hardware
- `M5Stack_ESP32_test` - Real ESP32 hardware
- `Nano_test` - Real Arduino Nano hardware

## Critical Testing Rules

### 1. Test Isolation (MANDATORY)
Each test MUST be completely isolated from other tests:

```cpp
test(TestSuite, TestName) {
    resetCounters(); // ALWAYS reset global counters first
    
    // Create LOCAL instances - never use global objects
    State state1("State1", onEnterCallback);
    SimpleFSM fsm; // Local FSM instance
    
    // Test logic here
}
```

**Why**: Global state pollution was the root cause of previous segmentation faults.

### 2. Arduino C++ Compatibility (MANDATORY)
Always use Arduino-compatible C++ constructs:

```cpp
// CORRECT - Use NULL for Arduino compatibility
Transition globalTrans(NULL, &target, event);
if (state == NULL) { ... }

// WRONG - nullptr may not be available in all Arduino environments
Transition globalTrans(nullptr, &target, event); // DON'T USE
if (state == nullptr) { ... } // DON'T USE
```

**Why**: Arduino uses embedded C++ which may not support all modern C++ features.

### 3. Timing in Timed Transition Tests
For tests involving `TimedTransition`, ALWAYS use small run intervals:

```cpp
// CORRECT - Use small interval for timed transitions
fsm.run(10); // 10ms interval

// WRONG - Default 1000ms interval will miss short timeouts
fsm.run(); // Uses 1000ms default, will miss 50ms timeouts
```

**Why**: The FSM only checks timed transitions when `run()` is called, and the default interval is 1000ms.

### 4. Counter Management
Use the provided callback counter system:

```cpp
// Global counters (already defined)
static int g_enter_count = 0;
static int g_state_count = 0;
static int g_exit_count = 0;
// etc.

// Always reset before each test
resetCounters();

// Use in assertions
assertEqual(g_enter_count, 1);
```

### 5. Memory Safety
- Always use **pointer arrays** for states: `State* states[] = { &state1, &state2 };`
- Never use value arrays: `State states[] = { ... }` (won't work with FSM.add())
- Check return values: `FSMError result = fsm.add(states, 2);`

### 6. Timing Loops for Timed Transitions
Use this pattern for testing timed transitions:

```cpp
bool transitioned = false;
for (int i = 0; i < MAX_ATTEMPTS && !transitioned; i++) {
    fsm.run(SMALL_INTERVAL); // e.g., 10ms
    if (fsm.getState() == &target_state) {
        transitioned = true;
    }
    delay(SMALL_DELAY); // e.g., 5ms
}
assertTrue(transitioned);
```

## Test Categories and Expectations

### StateManagement Tests
- Test all state callbacks: `on_enter`, `on_state`, `on_exit`
- Test final states and FSM termination
- Test state properties and setters
- Test state ID assignment and uniqueness

### TransitionTests  
- Test event-driven transitions with various event IDs
- Test guard conditions that allow/block transitions
- Test transition callbacks (`on_run_cb`)
- Test invalid event handling
- Test multiple transitions from same state

### TimedTransitions Tests
- **CRITICAL**: Use `fsm.run(10)` with 10ms interval for timing tests
- Test single timed transitions with callbacks
- Test multiple overlapping timed transitions
- Test timed transitions with guard conditions
- Test timer reset behavior on state changes

### GlobalTransitions Tests
- Test modern helper API: `addGlobalTransition()`, `addGlobalTimedTransition()`
- Test traditional approach: `Transition(NULL, &target, event)` (NOTE: Use NULL, not nullptr for Arduino compatibility)
- Test global transitions work from multiple states
- Test global vs local transition interactions

### ErrorHandling Tests
- Test all `FSMError` enum values
- Test parameter validation (null pointers, zero sizes)
- Test memory limits (MAX_STATES, MAX_TRANSITIONS, MAX_TIMED_TRANSITIONS)
- Test error string retrieval
- Test duplicate detection

### AdvancedFeatures Tests
- Test `fsm.reset()` returns to initial state

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LennartHennigs/SimpleFSM](https://github.com/LennartHennigs/SimpleFSM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
