---
trigger: always_on
description: Generates: `transformDataStringIntDictionaryStringArrayInt`
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Swift-Spyable is a Swift macro library that generates spy/mock classes for protocols. It replaces manual test double creation with automated, type-safe spy generation using Swift macros.

## Common Commands

### Building and Testing
- `swift build` - Build the package
- `swift test` - Run all tests
- `swift test -Xswiftc -Xfrontend -Xswiftc -dump-macro-expansions --enable-code-coverage` - Run tests with coverage and macro expansion dumps
- `swift test --filter TestName` - Run specific test

### Code Formatting
- `swift format --recursive --in-place ./Package.swift ./Sources ./Tests ./Examples` - Format all code (automatically done by CI)

### Platform-Specific Testing
- macOS: Use Xcode 15.4+ or 16.2+
- Linux: Requires Swift 5.9+
- Run Examples: `cd Examples && swift test`

## Architecture

The codebase follows a clear separation between public API and implementation:

### Core Structure
- `Sources/Spyable/` - Public API (`@Spyable` macro)
- `Sources/SpyableMacro/` - Macro implementation
  - `Macro/SpyableMacro.swift` - Main macro entry point
  - `Factories/` - Code generation logic split by concern
    - `VariablePrefixFactory.swift` - Generates unique variable prefixes with polymorphism support
  - `Extractors/` - Protocol syntax extraction
  - `Extensions/` - SwiftSyntax utilities
  - `Helpers/` - Utility classes
    - `TypeSanitizer.swift` - Type name sanitization for variable naming
  - `Diagnostics/` - Error handling

### Key Design Patterns
1. **Factory Pattern**: Each aspect of spy generation (methods, properties, call tracking) has its own factory
2. **Visitor Pattern**: Uses SwiftSyntax visitors to traverse and analyze protocol declarations
3. **Builder Pattern**: Constructs spy classes incrementally through multiple factories

### Generated Spy Structure
For a protocol `MyProtocol`, the macro generates `MyProtocolSpy` with:
- `{method}Called` - Bool tracking if method was called
- `{method}CallsCount` - Int counting method calls
- `{method}ReceivedArguments` - Tuple of last received arguments
- `{method}ReceivedInvocations` - Array of all invocations
- `{method}Closure` - Optional closure for stubbing behavior
- `{method}ReturnValue` - Stubbed return value (non-void methods)
- `{method}ThrowableError` - Error to throw (throwing methods)

## Polymorphism Support

Swift-Spyable automatically handles polymorphic functions (methods with the same name but different parameter or return types) by generating descriptive variable names that include type information. This ensures each method overload gets unique spy variables without naming conflicts.

### Implementation Architecture

The polymorphism detection system consists of three main components:

#### 1. VariablePrefixFactory
Located in `Sources/SpyableMacro/Factories/VariablePrefixFactory.swift`, this factory generates unique textual representations for function declarations:

- **Non-descriptive mode** (default): Uses function name + parameter names (e.g., `displayTextName`)
- **Descriptive mode** (polymorphism detected): Includes parameter and return types (e.g., `displayTextStringNameStringString`)

The factory automatically switches to descriptive mode when `SpyFactory` detects multiple functions with the same non-descriptive prefix.

#### 2. TypeSanitizer Helper
Located in `Sources/SpyableMacro/Helpers/TypeSanitizer.swift`, this utility sanitizes Swift type names for use in variable identifiers:

- Removes forbidden characters: `[`, `]`, `<`, `>`, `(`, `)`, `,`, ` `, `-`, `&`, `:`
- Handles optionals: `String?` becomes `OptionalString`, `String??` becomes `OptionalOptionalString`
- Processes function attributes: `@escaping` becomes `escaping`, `@Sendable` becomes `Sendable`
- Sanitizes complex nested types like `[String: [Int]]` → `StringInt`

#### 3. SpyFactory Integration
The main `SpyFactory` orchestrates polymorphism detection by:

1. Pre-scanning all functions to build a frequency map of non-descriptive prefixes
2. Identifying functions that would have naming conflicts (frequency > 1)
3. Automatically enabling descriptive mode for conflicting functions
4. Generating unique variable names for each method overload

### Polymorphism Examples

Given these polymorphic methods:
```swift
protocol DisplayService {
    func display(text: Int, name: String)
    func display(text: String, name: String) 
    func display(text: String, name: String) -> String
}
```

Swift-Spyable generates:
```swift
class DisplayServiceSpy: DisplayService {
    // For display(text: Int, name: String)
    var displayTextIntNameStringCalled = false
    var displayTextIntNameStringCallsCount = 0
    // ... other spy variables
    
    // For display(text: String, name: String)
    var displayTextStringNameStringCalled = false
    var displayTextStringNameStringCallsCount = 0
    // ... other spy variables
    
    // For display(text: String, name: String) -> String
    var displayTextStringNameStringStringCalled = false
    var displayTextStringNameStringStringCallsCount = 0
    var displayTextStringNameStringStringReturnValue: String!
    // ... other spy variables
}
```

### Testing Strategy for Polymorphic Functions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Matejkob/swift-spyable](https://github.com/Matejkob/swift-spyable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
