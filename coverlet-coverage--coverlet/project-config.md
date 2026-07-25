---
trigger: always_on
description: This is a .NET based repository that contains the coverlet projects for code coverage collection. Please follow these guidelines when contributing:
---

# GitHub Copilot Instructions

This is a .NET based repository that contains the coverlet projects for code coverage collection. Please follow these guidelines when contributing:

## General Guidelines
- Always ask before creating any documentation files (e.g., `.md`, `.txt`, `.rst`). Prioritize code and test changes only.
- Create a single, well-organized proposal or resolution document per issue, located in `Documentation/Plans/`. This folder is Git-ignored and local-only; documents are NOT uploaded to GitHub.
- Stop deep analysis once effort/usage for a topic reaches about 60%, and keep investigation focused and efficient.

## Code Standards

You MUST follow all code-formatting and naming conventions defined in [`.editorconfig`](../.editorconfig).

In addition to the rules enforced by `.editorconfig`, you SHOULD:

- Favor style and conventions that are consistent with the existing codebase.
- Prefer file-scoped namespace declarations and single-line using directives.
- Ensure that the final return statement of a method is on its own line.
- Use pattern matching and switch expressions wherever possible.
- Use `nameof` instead of string literals when referring to member names.
- Always use `is null` or `is not null` instead of `== null` or `!= null`.
- Trust the C# null annotations and don't add null checks when the type system says a value cannot be null.
- Prefer `?.` if applicable (e.g. `scope?.Dispose()`).
- Use `ObjectDisposedException.ThrowIf` where applicable.
- Respect StyleCop.Analyzers rules, in particular:
  - SA1028: Code must not contain trailing whitespace
  - SA1316: Tuple element names should use correct casing
  - SA1518: File is required to end with a single newline character
- Any update for pattern matching `or` must not break the already-correct `text == "hello" || text == "world"` operator scenario.

## Testing Guidelines

- Tests for coverlet MUST use xunit.v3.
- Overall code test coverage for shipping projects (coverlet nuget packages) shall not be below 90%.
- Aim for 100% branch coverage in the coverage report for the pattern matching `or` case (e.g., `text is "hello" or "world"`). Tests may inspect current raw branch behavior for diagnosis but should ultimately validate the correct 100% reported branch coverage outcome.

## Testing Requirements

- **Location**: `test/`
- **namespace** for tests must have the same root namespace as the code under test, with an additional `.Tests` suffix.
- **Coverage Requirement**: Overall 90% test coverage for all modules.
- **Best Practices**:
  - Follow existing test patterns.
  - New test samples for coverlet.core.coverage.tests must be added at the end of the source file, and existing tests should not be rearranged because line numbers are used within tests.

### Unit Test Guidelines (Critical Rules)

#### File System Abstraction (Mandatory)

**Unit tests MUST NOT use the file system directly.** Instead:

1. **Use `IFileSystem` abstraction** - All file operations must go through `Coverlet.Core.Abstractions.IFileSystem`.
2. **Mock file system operations** - Use `Mock<IFileSystem>` in tests.
3. **Simulate paths** - Use fake/simulated paths (e.g., `/fake/path/test.dll`) instead of creating real files.
4. **Verify behavior** - Verify that code calls the abstracted methods correctly.

**Examples:**

❌ **INCORRECT** - Direct file system usage:

```cs
// BAD - Creates real files and directories
File.WriteAllText("report.json", content); Directory.CreateDirectory("reports");
bool exists = File.Exists("test.dll");
```

✅ **CORRECT** - Mock file system:

```cs
// GOOD - Uses mocked abstraction with simulated paths
var mockFileSystem = new Mock<IFileSystem>();
mockFileSystem.Setup(x => x.Exists("/fake/path/test.dll")).Returns(true);
mockFileSystem.Setup(x => x.Exists("/fake/reports")).Returns(true);
mockFileSystem.Setup(x => x.WriteAllText(It.IsAny<string>(), It.IsAny<string>()));
// Verify the mock was called correctly
mockFileSystem.Verify(x => x.WriteAllText(It.Is<string>(path => path.EndsWith("report.json")), It.IsAny<string>()), Times.Once);
```

### Moq Testing Rules (Critical - Prevents Runtime Errors)

**NEVER use extension methods in Moq `Setup()` or `Verify()` calls.**

Extension methods are static methods that cannot be intercepted by Moq. Using them will result in a `System.NotSupportedException` at runtime with the message: "Unsupported expression: Extension methods may not be used in setup / verification expressions."

**Common Extension Methods That CANNOT Be Mocked:**
- `ILogger.LogInformation()`, `LogWarning()`, `LogError()`, `LogDebug()`, `LogTrace()`, `LogCritical()`
- LINQ extension methods on interfaces (e.g., `IEnumerable<T>.Where()`, `First()`, `Any()`)
- Any custom extension methods

**Solution:** Mock the underlying interface method that the extension method calls internally.

#### Example: Mocking ILogger

❌ **INCORRECT** - Will throw `NotSupportedException`:

```cs
// This will FAIL at runtime
_mockLogger.Verify(x => x.LogInformation(It.IsAny<string>()), Times.Once);
_mockLogger.Verify(x => x.LogInformation(It.Is<string>(s => s.Contains("json"))), Times.Once);
_mockLogger.Setup(x => x.LogWarning(It.IsAny<string>()));
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coverlet-coverage/coverlet](https://github.com/coverlet-coverage/coverlet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
