---
trigger: always_on
description: Detester is a .NET library for building deterministic and reliable tests for AI applications. It provides a fluent builder API for testing AI responses with support for OpenAI, Azure OpenAI, and custom IChatClient implementations.
---

# GitHub Copilot Instructions for Detester

## Project Overview

Detester is a .NET library for building deterministic and reliable tests for AI applications. It provides a fluent builder API for testing AI responses with support for OpenAI, Azure OpenAI, and custom IChatClient implementations.

## Core Technologies

- **.NET 10.0**: Current target framework
- **Microsoft.Extensions.AI**: Core abstraction for AI integrations
- **OpenAI SDK**: For OpenAI integration
- **Azure.AI.OpenAI**: For Azure OpenAI integration
- **xUnit**: Testing framework
- **StyleCop**: Code style enforcement

## Code Style and Conventions

### General Guidelines

- Follow C# naming conventions (PascalCase for public members, camelCase for private fields)
- Use XML documentation comments for all public APIs
- Follow StyleCop rules configured in `stylecop.json`
- Use nullable reference types appropriately
- Prefer explicit over implicit typing for clarity
- Use modern language features only when they improve clarity (avoid novelty for its own sake)

### API Design

- Use fluent builder pattern for API methods (return `this` or interface type)
- Validate input parameters and throw appropriate exceptions with descriptive messages
- Use `ArgumentNullException` for null reference parameters
- Use `ArgumentException` for invalid parameter values
- Throw `DetesterException` for domain-specific errors
- Avoid exposing mutable internal collections

### Testing

- Write comprehensive unit tests using xUnit
- Use `[Fact]` for simple tests, `[Theory]` with `[InlineData]` for parameterized tests
- Follow Arrange-Act-Assert pattern (without comments; structure code accordingly)
- Mock external dependencies (use `MockChatClient` for `IChatClient`)
- Test both success and failure scenarios
- Prefer deterministic test data; avoid reliance on network unless explicitly integration

## Project Structure

```
/src
  /Detester                    - Main library with implementations
  /Detester.Abstraction        - Interfaces and base types
/test
  /Detester.Tests              - Unit tests
```

## Key Classes and Interfaces

### Core Components

- `IDetesterBuilder`: Main interface for fluent API
- `DetesterBuilder`: Implementation of the builder pattern
- `DetesterFactory`: Factory methods for creating builder instances
- `DetesterOptions`: Configuration options
- `DetesterException`: Custom exception type

### Factory Methods

- `CreateWithOpenAI(apiKey, modelName)`: Create builder for OpenAI
- `CreateWithAzureOpenAI(endpoint, apiKey, deploymentName)`: Create builder for Azure OpenAI
- `Create(options)`: Create builder from options
- `Create(chatClient)`: Create builder with custom chat client

### Builder Methods

- `WithPrompt(prompt)`: Add a single prompt
- `WithPrompts(params prompts)`: Add multiple prompts
- `ShouldContainResponse(expectedText)`: Add assertion for response content
- `AssertAsync(cancellationToken)`: Execute and validate

## Implementation Guidelines

### When Adding New Features

1. Start with interface definition in `Detester.Abstraction`
2. Add XML documentation comments
3. Implement in `Detester` project
4. Add factory methods if needed
5. Write comprehensive tests
6. Update README.md with examples
7. Maintain backward compatibility of fluent chains

### When Fixing Bugs

1. Write a failing test that reproduces the issue
2. Fix the implementation
3. Ensure all existing tests still pass
4. Update documentation if needed
5. Add regression test coverage

### Building and Testing

```bash
# Restore dependencies
dotnet restore

# Build the solution
dotnet build

# Run all tests
dotnet test
```

## Common Patterns

### Creating a New Builder Method

```csharp
/// <summary>
/// Description of what the method does.
/// </summary>
/// <param name="paramName">Parameter description.</param>
/// <returns>The builder instance for method chaining.</returns>
public IDetesterBuilder MethodName(string paramName)
{
    if (string.IsNullOrWhiteSpace(paramName))
    {
        throw new ArgumentException("Parameter cannot be null or whitespace.", nameof(paramName));
    }

    // Implementation logic
    
    return this;
}
```

### Error Handling

- Validate inputs early and fail fast
- Use descriptive error messages
- Include parameter names in exceptions
- Don't catch exceptions unless you can handle them meaningfully

## Dependencies

- Keep dependencies minimal and up to date
- Only add new dependencies when absolutely necessary
- Prefer `Microsoft.Extensions.AI` abstractions over direct SDK usage when possible
- Review dependency licensing before inclusion

## Documentation

- Update README.md for any user-facing changes
- Include code examples in documentation
- Keep API reference section synchronized with actual API
- Document breaking changes clearly

## Example Usage Pattern

```csharp
// Typical test pattern
await builder
    .WithPrompt("User prompt")
    .ShouldContainResponse("expected text")
    .AssertAsync();

// Multiple prompts and assertions
await builder
    .WithPrompts("Prompt 1", "Prompt 2")
    .ShouldContainResponse("keyword1")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sa-es-ir/detester](https://github.com/sa-es-ir/detester) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
