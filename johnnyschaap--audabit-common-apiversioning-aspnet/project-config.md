---
trigger: always_on
description: > **Note**: This is a **library-specific** template for Audabit.Common.* NuGet packages. These are infrastructure libraries, NOT Web API projects.
---

# Development Guidelines & Code Patterns - Audabit Common Library

> **Note**: This is a **library-specific** template for Audabit.Common.* NuGet packages. These are infrastructure libraries, NOT Web API projects.

This file serves as a reference for code generation patterns used in Audabit Common libraries. Follow these conventions when generating extension methods, handlers, settings, validators, and other library code.

---

## Core Design Principles

### DRY Principle (Don't Repeat Yourself)
**CRITICAL**: Always avoid code duplication. This is a fundamental requirement for maintainable, testable, and scalable code.

#### When to Apply DRY
- **Method Overloads**: More specific overloads should delegate to more general overloads
  ```csharp
  // ✅ GOOD: Generic overload delegates to specific overload
  public static IServiceCollection AddObservability<TSettings>(
      this IServiceCollection services,
      IConfigurationSection configurationSection,
      Func<TSettings?, string?> serviceNameSelector) where TSettings : class
  {
      ArgumentNullException.ThrowIfNull(services);
      ArgumentNullException.ThrowIfNull(configurationSection);
      ArgumentNullException.ThrowIfNull(serviceNameSelector);

      var settings = configurationSection.Get<TSettings>();
      var serviceName = serviceNameSelector(settings) ?? "UnknownService";

      return services.AddObservability(serviceName); // Delegates to core method
  }

  // ❌ BAD: Duplicate registration logic
  public static IServiceCollection AddObservability<TSettings>(...)
  {
      // ... validation ...
      services.AddSingleton(typeof(IEmitter<>), typeof(Emitter<>)); // DUPLICATE
      LoggingEvent.SetServiceName(serviceName ?? "UnknownService");  // DUPLICATE
      return services;
  }
  ```

- **Configuration Methods**: Extract common configuration, then compose
  ```csharp
  // ✅ GOOD: Clear providers then delegate
  public static IServiceCollection UseJsonConsoleLogging(
      this IServiceCollection services,
      Action<ILoggingBuilder>? configureLogging = null)
  {
      ArgumentNullException.ThrowIfNull(services);
      services.AddLogging(builder => builder.ClearProviders());
      return services.AddJsonConsoleLogging(configureLogging); // Reuses core config
  }

  // ❌ BAD: Duplicate JSON console configuration
  public static IServiceCollection UseJsonConsoleLogging(...)
  {
      services.AddLogging(builder =>
      {
          builder.ClearProviders();
          builder.AddJsonConsole(options => { /* DUPLICATE */ });
          configureLogging?.Invoke(builder);
      });
      return services;
  }
  ```

- **Shared Logic**: Extract to helper methods or base classes
- **Constant Values**: Define once, reference everywhere
- **Validation Rules**: Centralize validation logic (FluentValidation)

#### Benefits of DRY
- **Single Source of Truth**: Changes happen in one place
- **Reduced Bugs**: Fix once, fixes everywhere
- **Easier Testing**: Test core logic once, not repeatedly
- **Better Maintainability**: Less code to understand and maintain
- **Consistency**: Behavior is uniform across all usages

#### DRY Detection Patterns
Watch for these signs of duplication:
- Copying and pasting code blocks
- Similar method implementations with minor variations
- Repeated conditional logic
- Identical configuration patterns
- Same validation rules in multiple places

**Rule**: If you find yourself writing the same code twice, extract it. If you find yourself writing similar code, abstract it.

---

## Global Usings Pattern

### CRITICAL: No Usings.cs File Required

**All projects** in the Audabit solution use **global usings defined in the .csproj file**, NOT in a separate `Usings.cs` file.

**Rule**: Never create a `Usings.cs` file. All global usings are declared in the project file.

**Template**:
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="AutoFixture" Version="4.18.1" />
    <PackageReference Include="AutoFixture.Xunit2" Version="4.18.1" />
    <PackageReference Include="NSubstitute" Version="5.3.0" />
    <PackageReference Include="Shouldly" Version="4.3.0" />
    <PackageReference Include="xunit" Version="2.9.2" />
  </ItemGroup>

  <!-- Global usings defined HERE, not in Usings.cs -->
  <ItemGroup>
    <Using Include="AutoFixture" />
    <Using Include="AutoFixture.Xunit2" />
    <Using Include="NSubstitute" />
    <Using Include="Shouldly" />
    <Using Include="Xunit" />
  </ItemGroup>
</Project>
```

**Why**:
- Single source of truth (no duplicate files across projects)
- Clear visibility in project file
- Consistent with modern .NET conventions
- Easier to maintain and understand

**Unit Test Projects**: Include AutoFixture, AutoFixture.Xunit2, NSubstitute, Shouldly, Xunit  
**Integration Test Projects**: Same as unit tests (consistent pattern)  
**Library Projects**: Minimal usings based on actual dependencies

---

## Library Structure

### Typical Audabit.Common.* Package Organization
```
Audabit.Common.{Feature}.AspNet/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JohnnySchaap/Audabit.Common.ApiVersioning.AspNet](https://github.com/JohnnySchaap/Audabit.Common.ApiVersioning.AspNet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
