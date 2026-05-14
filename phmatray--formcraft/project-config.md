---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

### Building the Project
```bash
# Restore dependencies and build
dotnet restore
dotnet build

# Build in Release mode
dotnet build --configuration Release

# Build with warnings as errors (for CI/CD validation)
dotnet build /p:TreatWarningsAsErrors=true

# Create local NuGet package
./pack-local.sh  # macOS/Linux - Creates packages in ./nupkg/
./pack-local.ps1 # Windows
```

### Running Tests
```bash
# Run all tests (600+ unit tests across 2 test projects)
dotnet test

# Run specific test project
dotnet test FormCraft.UnitTests/FormCraft.UnitTests.csproj
dotnet test FormCraft.ForMudBlazor.UnitTests/FormCraft.ForMudBlazor.UnitTests.csproj

# Run tests with coverage
dotnet test --collect:"XPlat Code Coverage"

# Run specific test class or method
dotnet test --filter "FullyQualifiedName~FormBuilderTests"
dotnet test --filter "DisplayName~Should_Build_Valid_Configuration"

# Run tests by category
dotnet test --filter "Category=Builder"
dotnet test --filter "Category=Renderer"
dotnet test --filter "Category=Security"
```

### Running the Demo Application
```bash
cd FormCraft.DemoBlazorApp
dotnet run
# Navigate to https://localhost:5001 (or http://localhost:5000)
```

### NUKE Build System
The project uses NUKE for sophisticated build automation:
```bash
# Run full build pipeline (macOS/Linux)
./build.sh

# Run full build pipeline (Windows)
./build.ps1

# Available NUKE targets:
# - Clean: Cleans build outputs
# - Restore: Restores NuGet packages
# - Compile: Builds the solution
# - Test: Runs all unit tests
# - Pack: Creates NuGet packages
# - Changelog: Generates changelog using git-cliff
```

## High-Level Architecture

### Solution Structure
```
FormCraft/                      # Core library (framework-agnostic)
├── Builders/                   # Fluent API builders
│   ├── FormBuilder.cs         # Main entry point
│   ├── FieldBuilder.cs        # Individual field configuration
│   └── FieldGroupBuilder.cs   # Field grouping and layout
├── Configuration/              # Configuration models
├── Rendering/                  # Rendering pipeline
│   ├── IFieldRenderer.cs      # Renderer contract
│   └── FieldRendererService.cs # Renderer registry
├── Validation/                 # Validation system
│   └── IFieldValidator.cs     # Validator contract
├── Security/                   # Security features (v2.0.0+)
│   ├── IEncryptionService.cs  # Field encryption
│   └── ICsrfTokenService.cs   # CSRF protection
└── Extensions/                 # Extension methods

FormCraft.ForMudBlazor/         # MudBlazor UI implementation
├── Renderers/                  # MudBlazor-specific renderers
└── Services/                   # UI framework services

FormCraft.DemoBlazorApp/        # Interactive demo application
FormCraft.UnitTests/            # Core library test suite (560+ tests)
FormCraft.ForMudBlazor.UnitTests/ # MudBlazor integration tests (47 tests)
build/                          # NUKE build automation
```

### Target Frameworks
- **net9.0** and **net10.0** - Multi-targeting for .NET 9 and .NET 10

### Core Design Patterns

#### 1. Fluent Builder Pattern (Primary Architecture)
The entire API is built around method chaining with immutable configuration:
```csharp
FormBuilder<TModel>.Create()
    .AddField(x => x.Property, field => field.ConfigureField())
    .AddFieldGroup(group => group.ConfigureGroup())
    .WithLayout(FormLayout.Grid)
    .WithSecurity(security => security.ConfigureSecurity())
    .Build() // Returns immutable IFormConfiguration<TModel>
```

**Key Builder Classes:**
- `FormBuilder<TModel>` - Root builder, entry point via `.Create()`
- `FieldBuilder<TModel, TValue>` - Configures individual fields
- `FieldGroupBuilder<TModel>` - Groups fields with layout options
- `SecurityBuilder<TModel>` - Security features configuration (encryption, CSRF, rate limiting)

#### 2. Strategy Pattern (Field Rendering)
Pluggable rendering system with type-based renderer selection:
```csharp
public interface IFieldRenderer
{
    bool CanRender(Type fieldType, IFieldConfiguration<object, object> field);
    RenderFragment Render<TModel>(IFieldRenderContext<TModel> context);
}
```

**Renderer Registration:**
- Default renderers registered in DI container
- Custom renderers via `.WithCustomRenderer()`
- Priority-based selection when multiple renderers match

#### 3. Command Pattern (Validation)
Async validation with command pattern:
```csharp
public interface IFieldValidator<TModel, TValue>
{
    Task<ValidationResult> ValidateAsync(TModel model, TValue value, IServiceProvider services);
}
```

**Built-in Validators:**
- `RequiredValidator<TModel, TValue>`
- `CustomValidator<TModel, TValue>`
- `AsyncValidator<TModel, TValue>`
- FluentValidation integration via `DynamicFormValidator`

#### 4. Observer Pattern (Field Dependencies)
Reactive field updates based on dependencies:
```csharp
.AddField(x => x.TotalPrice)
    .DependsOn(x => x.Quantity, x => x.Price)
    .WithValueProvider((model, services) => model.Quantity * model.Price)
    .WithVisibilityProvider(model => model.Quantity > 0)
```

**Dependency Types:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phmatray/FormCraft](https://github.com/phmatray/FormCraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
