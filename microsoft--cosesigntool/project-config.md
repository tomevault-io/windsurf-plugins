---
trigger: always_on
description: This file ensures GitHub Copilot follows the repository's coding standards as defined in .editorconfig and established patterns.
---

# CoseSignTool Coding Standards for GitHub Copilot

This file ensures GitHub Copilot follows the repository's coding standards as defined in .editorconfig and established patterns.

## Code Generation Preferences

### File Headers
- Always include the Microsoft copyright header at the top of all C# files:
```csharp
// Copyright (c) Microsoft Corporation.
// Licensed under the MIT License.
```

### Namespace and Using Directives
- Use file-scoped namespaces (C# 10+ feature): `namespace MyNamespace;`
- Place using directives inside the namespace
- Sort System directives first, then others alphabetically
- Do not separate import directive groups with blank lines
- Follow namespace-to-folder structure matching

### Naming Conventions
- **Constants**: PascalCase (e.g., `DefaultStoreName`)
- **Static private fields**: PascalCase (e.g. `StaticStoreName`)
- **Private/internal instance fields**: camelCase (e.g., `commands`)
- **Public properties/methods**: PascalCase
- **Local variables**: camelCase
- **Parameters**: camelCase

### Code Style Preferences
- **Braces**: Always use braces for control statements (enforced as error)
- **var usage**: Avoid `var` - use explicit types for clarity
- **this qualifier**: useh `this.` so that it's clear when member parameters or state is being modified
- **Predefined types**: Use predefined types (`int`, `string`) over .NET types (`Int32`, `String`)
- **Null checking**: Prefer `is null` over `== null`
- **Object initialization**: Prefer object and collection initializers
- **String interpolation**: Use simplified interpolation when possible

### Expression Preferences
- **Expression-bodied members**: 
  - Use for properties, indexers, and accessors when appropriate
  - Avoid for methods, constructors, and operators (prefer block bodies)
- **Pattern matching**: Prefer pattern matching over `is` with cast checks
- **Target-typed new**: Use when type is apparent (e.g., `List<string> items = new();`)
- **Using statements**: Prefer simple using statements over using blocks

### Formatting Rules
- **Indentation**: **ALWAYS** use 4 spaces for indentation. **NEVER** use tabs.
  - This applies to ALL file types: .cs, .csproj, .xml, .json, .md files
  - Convert any existing tabs to 4 spaces
  - Configure your editor to show whitespace to avoid accidental tab usage
- **End of line**: CRLF (Windows line endings)
- **Final newline**: Do not insert final newline
- **Trim whitespace**: Always trim trailing whitespace
- **New lines**: 
  - Opening braces on new line for all constructs
  - `else`, `catch`, `finally` on new lines
  - Members in object initializers on new lines

### Space Preferences
- No space after casts: `(int)value`
- Space after keywords: `if (condition)`
- Space around binary operators: `a + b`
- Space after commas: `Method(a, b, c)`
- Space around colons in inheritance: `class Derived : Base`
- No space before dots: `object.Property`
- No space in empty parentheses: `Method()`

### Modifier Order
Follow this order: `public`, `private`, `protected`, `internal`, `static`, `extern`, `new`, `virtual`, `abstract`, `sealed`, `override`, `readonly`, `unsafe`, `volatile`, `async`

### Error Handling and Diagnostics
- Null reference warnings are suggestions, not errors
- Unused parameters should be flagged
- Platform compatibility should be validated
- File headers are required (enforced as error)
- Missing braces are errors
- Unused private members are errors

### Plugin Project-Specific Patterns
- **Plugin naming**: Use `.Plugin.csproj` suffix for auto-packaging
- **Plugin Assembly naming**: Use `.Plugin.dll` suffix for runtime discovery
- **Plugin Exit codes**: Use the `PluginExitCode` enum for plugin commands
- **Async patterns**: Always use `CancellationToken` parameters in async methods
- **Plugin Interface implementation**: Implement plugin interfaces (`ICoseSignToolPlugin`, `IPluginCommand`)
- **Plugin Error handling**: Use appropriate exit codes and console error output

### Documentation
- Use XML documentation comments for public APIs
- Include parameter descriptions and return value documentation
- Use `<summary>`, `<param>`, `<returns>` tags appropriately
- Where possible provide appropriate `<example>` tags for different behaviors
- Document exceptions with `<exception>` tags

### Testing Patterns
- Use descriptive test method names
- Follow Arrange-Act-Assert pattern
- Use meaningful assertions with clear error messages
- Include both positive and negative test cases
- Cover as close to 100% of blocks as posible without overgenerating test cases
- Prefer NUnit tests with the new `Assert.That` syntax when writing new tests
- Prefer data-driven test cases when possible
- Prefer test local state over shared state to enable parallel execution
- Leverage maintainability patterns in test code by creating shared functions for common code

### Plugin Development Guidelines
- Implement `PluginCommandBase` for command implementations
- Use proper dependency injection patterns
- Handle configuration through `IConfiguration`
- Implement proper cancellation token support

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/CoseSignTool](https://github.com/microsoft/CoseSignTool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
