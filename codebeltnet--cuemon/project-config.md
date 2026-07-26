---
trigger: always_on
description: Writing Unit Tests in Cuemon
---


# Writing Unit Tests in Cuemon
This document provides instructions for writing unit tests in the Cuemon codebase. Please follow these guidelines to ensure consistency and maintainability.

## 1. Base Class

**Always inherit from the `Test` base class** for all unit test classes.  
This ensures consistent setup, teardown, and output handling across all tests.

> Important: Do NOT add `using Xunit.Abstractions`. xUnit v3 no longer exposes that namespace; including it is incorrect and will cause compilation errors. Use the `Codebelt.Extensions.Xunit` Test base class and `using Xunit;` as shown in the examples below. If you need access to test output, rely on the Test base class (which accepts the appropriate output helper) rather than importing `Xunit.Abstractions`.

```csharp
using Codebelt.Extensions.Xunit;
using Xunit;

namespace Your.Namespace
{
    public class YourTestClass : Test
    {
        public YourTestClass(ITestOutputHelper output) : base(output)
        {
        }

        // Your tests here
    }
}
```

## 2. Test Method Attributes

- Use `[Fact]` for standard unit tests.
- Use `[Theory]` with `[InlineData]` or other data sources for parameterized tests.

## 3. Naming Conventions

- **Test classes**: End with `Test` (e.g., `DateSpanTest`).
- **Test methods**: Use descriptive names that state the expected behavior (e.g., `ShouldReturnTrue_WhenConditionIsMet`).

## 4. Assertions

- Use `Assert` methods from xUnit for all assertions.
- Prefer explicit and expressive assertions (e.g., `Assert.Equal`, `Assert.NotNull`, `Assert.Contains`).

## 5. File and Namespace Organization

- Place test files in the appropriate test project and folder structure.
- Use namespaces that mirror the source code structure. The namespace of a test file MUST match the namespace of the System Under Test (SUT). Do NOT append ".Tests", ".Benchmarks" or similar suffixes to the namespace. Only the assembly/project name should indicate that the file is a test/benchmark (for example: Cuemon.Foo.Tests assembly, but namespace Cuemon.Foo).
  - Example: If the SUT class is declared as:
    ```csharp
    namespace Cuemon.Foo.Bar
    {
        public class Zoo { /* ... */ }
    }
    ```
    then the corresponding unit test class must use the exact same namespace:
    ```csharp
    namespace Cuemon.Foo.Bar
    {
        public class ZooTest : Test { /* ... */ }
    }
    ```
  - Do NOT use:
    ```csharp
    namespace Cuemon.Foo.Bar.Tests { /* ... */ } // ❌
    namespace Cuemon.Foo.Bar.Benchmarks { /* ... */ } // ❌
    ```
- The unit tests for the Cuemon.Foo assembly live in the Cuemon.Foo.Tests assembly.
- The functional tests for the Cuemon.Foo assembly live in the Cuemon.Foo.FunctionalTests assembly.
- Test class names end with Test and live in the same namespace as the class being tested, e.g., the unit tests for the Boo class that resides in the Cuemon.Foo assembly would be named BooTest and placed in the Cuemon.Foo namespace in the Cuemon.Foo.Tests assembly.
- Modify the associated .csproj file to override the root namespace so the compiled namespace matches the SUT. Example:
  ```xml
  <PropertyGroup>
    <RootNamespace>Cuemon.Foo</RootNamespace>
  </PropertyGroup>
  ```
- When generating test scaffolding automatically, resolve the SUT's namespace from the source file (or project/assembly metadata) and use that exact namespace in the test file header.

- Notes:
  - This rule ensures type discovery and XML doc links behave consistently and reduces confusion when reading tests.
  - Keep folder structure aligned with the production code layout to make locating SUT <-> test pairs straightforward.

## 6. Example Test

```csharp
using System;
using System.Globalization;
using Codebelt.Extensions.Xunit;
using Xunit;

namespace Cuemon
{
    /// <summary>
    /// Tests for the <see cref="DateSpan"/> class.
    /// </summary>
    public class DateSpanTest : Test
    {
        public DateSpanTest(ITestOutputHelper output) : base(output)
        {
        }

        [Fact]
        public void Parse_ShouldGetOneMonthOfDifference_UsingIso8601String()
        {
            var start = new DateTime(2021, 3, 5).ToString("O");
            var end = new DateTime(2021, 4, 5).ToString("O");

            var span = DateSpan.Parse(start, end);

            Assert.Equal("0:01:31:00:00:00.0", span.ToString());
            Assert.Equal(0, span.Years);
            Assert.Equal(1, span.Months);
            Assert.Equal(31, span.Days);
            Assert.Equal(0, span.Hours);
            Assert.Equal(0, span.Minutes);
            Assert.Equal(0, span.Seconds);
            Assert.Equal(0, span.Milliseconds);

            Assert.Equal(0.08493150684931507, span.TotalYears);
            Assert.Equal(1, span.TotalMonths);
            Assert.Equal(31, span.TotalDays);
            Assert.Equal(744, span.TotalHours);
            Assert.Equal(44640, span.TotalMinutes);
            Assert.Equal(2678400, span.TotalSeconds);
            Assert.Equal(2678400000, span.TotalMilliseconds);

            Assert.Equal(6, span.GetWeeks());
            Assert.Equal(-1566296493, span.GetHashCode());

            TestOutput.WriteLine(span.ToString());
        }
    }
}
```

## 7. Additional Guidelines

- Keep tests focused and isolated.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codebeltnet/cuemon](https://github.com/codebeltnet/cuemon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
