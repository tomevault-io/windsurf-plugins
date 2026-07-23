---
trigger: always_on
description: This file provides specific guidance for AI assistants (especially Claude Code) when working with the Button2 test suite.
---

# Button2 Test Suite - AI Assistant Guidelines

This file provides specific guidance for AI assistants (especially Claude Code) when working with the Button2 test suite.

## Test Suite Overview

The Button2 test suite consists of **91 comprehensive tests** organized into 6 test suites:

1. **test_basics** (10 tests) - Button initialization, configuration, and fundamental operations
2. **test_clicks** (20 tests) - Click detection (single, double, triple, long, patterns, bounce simulation)
3. **test_callbacks** (17 tests) - All event handler callbacks, including context retrieval
4. **test_states** (23 tests) - State management, queries, timing edge cases, read() contract
5. **test_configuration** (17 tests) - Settings management, context, and property validation
6. **test_multiple** (12 tests) - Multiple button scenarios and interactions

## Testing Architecture

### Simulated Pin State Approach

**DO NOT use MockHardware** - it was removed from the library. Instead, use the **simulated pin state** approach:

```cpp
// Global simulated pin state
static uint8_t simulatedPinState = HIGH;

// Custom state function
uint8_t getSimulatedPinState() {
  return simulatedPinState;
}

// CRITICAL: Initialize in correct order
Button2 createTestButton() {
  Button2 button;

  // 1. Set simulated state FIRST
  simulatedPinState = HIGH;

  // 2. Set state function SECOND
  button.setButtonStateFunction(getSimulatedPinState);

  // 3. Initialize button LAST
  button.begin(BUTTON_PIN, INPUT_PULLUP, true);

  return button;
}
```

## Critical Testing Rules

### 1. Test Isolation (MANDATORY)

Each test MUST be completely independent:

```cpp
test(test_suite, test_name) {
    resetHandlerVars(); // ALWAYS reset global state first

    // Create LOCAL button instance
    Button2 button = createTestButton();
    button.resetPressedState();

    // Test logic here
}
```

**Why**: Global state pollution causes test interference and false failures.

### 2. Button Initialization Pattern (MANDATORY)

**ALWAYS** use this exact pattern for test button creation:

```cpp
Button2 createTestButton() {
  Button2 button;

  // CRITICAL ORDER:
  simulatedPinState = !BUTTON_ACTIVE;              // 1. State first
  button.setButtonStateFunction(getSimulatedPinState);  // 2. Function second
  button.begin(BUTTON_PIN, BUTTON_MODE, BUTTON_ACTIVE == LOW);  // 3. Init last

  return button;
}
```

### 3. Button Simulation Pattern

Use helper functions for button interactions:

```cpp
void click(Button2& button, unsigned long duration) {
  // Press
  simulatedPinState = BUTTON_ACTIVE;
  button.loop();

  // Hold with periodic loop calls
  unsigned long startTime = millis();
  while (millis() < startTime + duration) {
    button.loop();
    delay(1);
  }

  // Release
  simulatedPinState = !BUTTON_ACTIVE;
  button.loop();
  delay(5);
  button.loop();
}
```

**Critical**: Call `button.loop()` repeatedly while button is pressed to allow debouncing and click detection.

To simulate sub-debounce mechanical bounce (noise resilience tests), use `injectBounce(button, durationMs=5)` — it briefly flips the pin state and pumps `loop()`, then restores it. Duration must be less than `DEBOUNCE_MS` to be filtered by the library.

### 4. Callback Testing Pattern

```cpp
// Global test flags
static bool g_clicked = false;
static bool g_pressed = false;

void resetHandlerVars() {
  g_clicked = false;
  g_pressed = false;
  // ... reset all flags
}

test(callbacks, click_handler) {
  resetHandlerVars();
  Button2 button = createTestButton();
  button.resetPressedState();

  button.setClickHandler([](Button2& btn) {
    g_clicked = true;
  });

  click(button, DEBOUNCE_MS);
  delay(BTN_DOUBLECLICK_MS);  // Wait for click to be reported
  button.loop();               // Process the timeout

  assertTrue(g_clicked);
}
```

## Common Pitfalls to Avoid

### 1. Initialization Order Error

❌ **WRONG**:
```cpp
Button2 button;
button.begin(PIN, MODE, true);
button.setButtonStateFunction(func);  // Too late - state already read!
```

✅ **CORRECT**:
```cpp
simulatedPinState = HIGH;
button.setButtonStateFunction(func);
button.begin(PIN, MODE, true);
```

### 2. Insufficient loop() Calls

❌ **WRONG**:
```cpp
simulatedPinState = LOW;
button.loop();
delay(100);  // Button state not processed!
simulatedPinState = HIGH;
button.loop();
```

✅ **CORRECT**:
```cpp
simulatedPinState = LOW;
button.loop();

while (millis() < endTime) {
  button.loop();  // Keep calling loop while pressed
  delay(1);
}

simulatedPinState = HIGH;
button.loop();
```

### 3. Missing Click Finalization

❌ **WRONG**:
```cpp
click(button, DEBOUNCE_MS);
// Check immediately - click not yet reported!
assertTrue(button.wasPressed());  // FAILS!
```

✅ **CORRECT**:
```cpp
click(button, DEBOUNCE_MS);
delay(BTN_DOUBLECLICK_MS);  // Wait for timeout
button.loop();               // Process timeout
assertTrue(button.wasPressed());  // NOW it works
```

### 4. Type Cast for assertEqual()

❌ **WRONG**:
```cpp
assertEqual(button.getDebounceTime(), 100);  // Ambiguous function call
```

✅ **CORRECT**:
```cpp
assertEqual(button.getDebounceTime(), (unsigned int)100);  // Clear type
```

### 5. Active HIGH/LOW Confusion


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LennartHennigs/Button2](https://github.com/LennartHennigs/Button2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
