---
trigger: always_on
description: - Follow KISS, DRY, SOLID, and YAGNI principles
---

# GitHub Copilot Instructions for Agent365-devTools

## Agent365 CLI Development Guidelines

### Engineering Principles
- Follow KISS, DRY, SOLID, and YAGNI principles
- Align CLI patterns with Azure CLI (`az`) conventions
- Keep changes minimal and focused on the problem at hand
- Reuse existing functions across commands; avoid duplication
- Critically review all changes before committing

### Code Organization
- Keep files small and focused
- Use constants for strings and values (see `Constants/` folder)
- Use `ErrorCodes.cs` and `ErrorMessages.cs` for error handling

### File Organization Guidelines

#### Multiple Classes Per File - Allowed Cases
- **Model/DTO files**: Related model classes, records, or structs can be grouped in a single file
- **Request/Response pairs**: API request and response classes for the same endpoint
- **Small supporting types**: Enums, small records, or helper classes closely tied to a main class
- **Nested or related interfaces**: Interface and its related types

#### When to Separate
- Large classes with significant logic
- Classes that could be reused independently
- Classes with different lifecycle or ownership

### Cross-Platform Compatibility
- All code must work on Windows, macOS, and Linux
- Test file paths, line endings, and shell commands for compatibility

### Testing Standards
- Use xUnit, FluentAssertions, and NSubstitute
- Focus on quality over quantity of tests
- Add regression tests for bug fixes
- Tests should verify CLI reliability
- **Tests must assert requirements, not implementation** — when a test is changed to match new code behavior (rather than to reflect a changed requirement), that is a red flag. A test that silently tracks whatever the code does provides no regression protection. If a test needs to be updated, explicitly document the requirement the new assertion encodes (use `because:` in FluentAssertions). If you cannot articulate a requirement reason, the test change should be questioned.
- **FluentAssertions `because:` is mandatory for non-obvious assertions** — any assertion on a URL structure, encoding format, security-sensitive behavior, or protocol requirement must include a `because:` clause explaining the invariant being enforced.
- **Dispose IDisposable objects properly**:
  - `HttpResponseMessage` objects created in tests must be disposed
  - Even in mock/test handlers, follow proper disposal patterns
  - Consider using `using` statements or ensure test handlers dispose responses
  - This applies to all `IDisposable` test objects to avoid analyzer warnings
- **Disable parallel execution for tests with shared state**:
  - Tests that modify environment variables must disable parallelization
  - Tests that access shared file system resources must run sequentially
  - Use `[CollectionDefinition("TestName", DisableParallelization = true)]` pattern
  - Add `[Collection("TestName")]` attribute to test class
  - **Pattern to follow**:
    ```csharp
    /// <summary>
    /// Tests must run sequentially because they modify environment variables.
    /// </summary>
    [CollectionDefinition("EnvTests", DisableParallelization = true)]
    public class EnvTestCollection { }

    [Collection("EnvTests")]
    public class MyTests
    {
        [Fact]
        public void Test_ModifiesEnvironmentVariable()
        {
            Environment.SetEnvironmentVariable("VAR", "value");
            try
            {
                // Test logic
            }
            finally
            {
                Environment.SetEnvironmentVariable("VAR", null);
            }
        }
    }
    ```

### Resource Management
- **Always dispose IDisposable objects** to prevent resource leaks:
  - `HttpResponseMessage` returned by `HttpClient.GetAsync()`, `PostAsync()`, etc. must be disposed
  - Use `using` statements for automatic disposal: `using var response = await httpClient.GetAsync(...);`
  - Even when checking `IsSuccessStatusCode` or reading content, wrap in `using`
  - This applies to all HTTP responses, streams, file handles, and other disposable resources
  - **Pattern to follow**:
    ```csharp
    // CORRECT - Dispose HttpResponseMessage
    using var response = await httpClient.GetAsync(url, cancellationToken);
    if (!response.IsSuccessStatusCode) { return null; }
    var content = await response.Content.ReadAsStringAsync(cancellationToken);

    // INCORRECT - Resource leak
    var response = await httpClient.GetAsync(url, cancellationToken);
    if (!response.IsSuccessStatusCode) { return null; }
    var content = await response.Content.ReadAsStringAsync(cancellationToken);
    ```

### Output and Logging
- No emojis or special characters in logs, output, or comments
- The output should be plain text, and display properly in windows, macOS, and Linux terminals
- Keep user-facing messages clear and professional
- Follow client-facing help text conventions

### Code Review Mindset
- Be cautious about deleting code; avoid `git restore` without review
- Do not create unnecessary documentation files
- For user-facing changes (features, bug fixes, behavioral changes): verify `CHANGELOG.md` has an entry in the `[Unreleased]` section

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/Agent365-devTools](https://github.com/microsoft/Agent365-devTools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
