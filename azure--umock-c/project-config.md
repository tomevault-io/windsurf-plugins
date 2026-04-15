---
trigger: always_on
description: umock-c is a C mocking library designed for Azure C projects, providing macro-based mock generation, expectation recording, and call verification. It enables sophisticated unit testing through compile-time mock generation and runtime call tracking with rich assertion capabilities.
---

# umock-c AI Coding Instructions

## Project Overview
umock-c is a C mocking library designed for Azure C projects, providing macro-based mock generation, expectation recording, and call verification. It enables sophisticated unit testing through compile-time mock generation and runtime call tracking with rich assertion capabilities.

## Core Architecture

### Mock Generation System
- **`MOCKABLE_FUNCTION(modifiers, result, function, ...)`**: Core macro that generates mockable function declarations and implementations
- **`ENABLE_MOCKS`**: Compilation flag that switches between production code (declarations only) and test code (full mock implementations)
- **`inc/umock_c/umock_c_prod.h`**: Production header that enables `MOCKABLE_FUNCTION` for both production and test builds
- **Macro Metaprogramming**: Heavily uses `macro-utils-c` for complex macro expansion and code generation

### Mock Control Flow
```c
// Production header (dependency.h)
#include "umock_c/umock_c_prod.h"
MOCKABLE_FUNCTION(, int, dependency_function, int, arg);

// Test file pattern
#define ENABLE_MOCKS
#include "dependency.h"  // Generates mock implementations
#undef ENABLE_MOCKS
#include "unit_under_test.h"  // Real implementation
```

### Call Recording & Verification
- **`STRICT_EXPECTED_CALL(call)`**: Records expected calls with argument validation
- **`EXPECTED_CALL(call)`**: Records expected calls without argument validation by default
- **Call Modifiers**: Chainable modifiers like `.SetReturn()`, `.IgnoreArgument()`, `.ValidateArgument()`
- **`umock_c_get_expected_calls()` / `umock_c_get_actual_calls()`**: String representations for test assertions

## Key Development Patterns

### Test Structure (AAA Pattern)
```c
TEST_FUNCTION(function_under_test_with_dependency_succeeds)
{
    // arrange
    STRICT_EXPECTED_CALL(dependency_function(42))
        .SetReturn(0);

    // act
    int result = function_under_test();

    // assert
    ASSERT_ARE_EQUAL(int, 0, result);
    ASSERT_ARE_EQUAL(char_ptr, umock_c_get_expected_calls(), umock_c_get_actual_calls());
}
```

### Mock Filtering (Performance Optimization)
```c
#define ENABLE_MOCK_FILTERING
#define please_mock_specific_function MOCK_ENABLED

#define ENABLE_MOCKS
#include "large_header.h"  // Only generates mocks for selected functions
#undef ENABLE_MOCKS
```

### Global Mock Configuration
```c
// Test suite setup
REGISTER_GLOBAL_MOCK_HOOK(malloc, my_malloc_hook);
REGISTER_GLOBAL_MOCK_RETURN(function_name, success_value);
REGISTER_GLOBAL_MOCK_FAIL_RETURN(function_name, failure_value);
```

### Call Modifiers (Fluent Interface)
```c
STRICT_EXPECTED_CALL(function(arg1, arg2, buffer))
    .SetReturn(42)
    .IgnoreArgument_arg1()
    .ValidateArgumentBuffer_buffer(expected_data, data_size)
    .CopyOutArgumentBuffer_buffer(output_data, output_size);
```

## Call Modifiers Reference

Call modifiers provide a fluent interface for configuring mock expectations. Each modifier returns a structure that allows chaining further modifiers. The last modifier in a chain overrides previous modifiers if any collision occurs.

### Argument Control Modifiers

#### IgnoreArgument Family
```c
// Ignore specific argument by name (preferred)
STRICT_EXPECTED_CALL(function(42, "test", buffer))
    .IgnoreArgument_arg1()     // Ignores first argument regardless of value
    .IgnoreArgument_buffer();  // Ignores buffer argument

// Ignore argument by index (0-based)
STRICT_EXPECTED_CALL(function(42, "test", buffer))
    .IgnoreArgument(0)         // Ignores first argument (42)
    .IgnoreArgument(2);        // Ignores third argument (buffer)

// Ignore all arguments at once
STRICT_EXPECTED_CALL(function(42, "test", buffer))
    .IgnoreAllArguments();     // All arguments ignored, only function name matters

// Automatic ignore with IGNORED_ARG
STRICT_EXPECTED_CALL(function(IGNORED_ARG, "test", IGNORED_ARG));  // Equivalent to IgnoreArgument on positions 0 and 2
```

#### ValidateArgument Family
```c
// Validate specific argument by name (overrides ignore)
EXPECTED_CALL(function(42, "test", buffer))     // EXPECTED_CALL ignores all by default
    .ValidateArgument_arg1()                    // But validate first argument
    .ValidateArgument_arg2();                   // And second argument

// Validate argument by index
EXPECTED_CALL(function(42, "test", buffer))
    .ValidateArgument(0)                        // Validate first argument
    .ValidateArgument(1);                       // Validate second argument
```

### Return Value Modifiers

#### SetReturn and SetFailReturn
```c
// Set successful return value
STRICT_EXPECTED_CALL(malloc(100))
    .SetReturn((void*)0x1234);                  // Return specific pointer

// Set both success and failure return values
STRICT_EXPECTED_CALL(network_send(data, size))
    .SetReturn(0)                               // Success case
    .SetFailReturn(-1);                         // Failure case for negative testing

// Capture the return value that was actually returned
int captured_return;
STRICT_EXPECTED_CALL(get_status())

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Azure) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
