---
trigger: always_on
description: This is a **C# source generator library** for ASP.NET Core that brings clean, class-based endpoints to Minimal APIs. The project targets **.NET 10.0** and uses modern C# features including file-scoped namespaces, collection expressions, and nullable reference types.
---

# GitHub Copilot Instructions for MinimalApi.Endpoints

## Project Overview

This is a **C# source generator library** for ASP.NET Core that brings clean, class-based endpoints to Minimal APIs. The project targets **.NET 10.0** and uses modern C# features including file-scoped namespaces, collection expressions, and nullable reference types.

### Repository Architecture

The repository is organized into three main categories:

#### **Core Library Projects**

1. **`src/IeuanWalker.MinimalApi.Endpoints/`** - Main library project
   - Interface definitions (`IEndpoint`, `IEndpointGroup`, `Validator`)
   - Extension methods and filters for consumers
   - Runtime utilities and base functionality
   - **No source generation logic** (handled separately in Generator project)

2. **`src/IeuanWalker.MinimalApi.Endpoints.Generator/`** - Source generator project
   - `EndpointGenerator.cs` - Main incremental source generator
   - Compile-time analysis of endpoint implementations
   - Generates registration extension methods (`AddEndpointsFrom{AssemblyName}()`, `MapEndpointsFrom{AssemblyName}()`)
   - Diagnostic reporting for configuration validation
   - Uses Roslyn's `IIncrementalGenerator` API for optimal performance

#### **Example & Demonstration**

3. **`example/ExampleApi/`** - Full-featured example application
   - Demonstrates all endpoint patterns and configurations
   - API versioning implementation
   - Scalar OpenAPI documentation integration
   - FluentValidation and DataAnnotations examples
   - Feature-based organization under `/Endpoints` folder

#### **Test Projects**

4. **`tests/IeuanWalker.MinimalApi.Endpoints.Tests/`** - Core library unit tests
5. **`tests/IeuanWalker.MinimalApi.Endpoints.Generator.Tests/`** - Source generator unit tests with snapshot testing
6. **`tests/ExampleApi.Tests/`** - Example API unit tests
7. **`tests/ExampleApi.IntegrationTests/`** - Comprehensive integration tests

**Testing Stack:**
- **xUnit** - Test framework
- **Shouldly** - Fluent assertions
- **Verify** (Verify.Xunit / Verify.SourceGenerators) - Snapshot testing for generated code
- **Microsoft.AspNetCore.Mvc.Testing** - Integration test hosting

## Core Architecture & Design Principles

### Zero-Runtime-Reflection Approach
- **Compile-time code generation** - All endpoint registration happens at build time
- **Performance optimized** - No reflection overhead during application startup or runtime
- **Incremental generation** - Only regenerates when relevant source code changes
- **Diagnostic validation** - Build-time validation of endpoint configurations with clear error messages

### Endpoint Interface System

The library provides four endpoint interfaces to cover all common scenarios:

```csharp
// Standard request/response endpoint
IEndpoint<TRequest, TResponse>

// Response-only endpoint (no request body)
IEndpointWithoutRequest<TResponse>

// Request-only endpoint (void return)
IEndpointWithoutResponse<TRequest>

// Simple endpoint (no request or response)
IEndpoint
```

**Required Implementation Pattern:**
```csharp
public class MyEndpoint : IEndpoint<RequestModel, ResponseModel>
{
    // 1. Static Configure method for route configuration
    public static void Configure(RouteHandlerBuilder builder)
    {
        builder.Get("/path").WithName("EndpointName");
    }
    
    // 2. Handle method with appropriate signature
    public async Task<ResponseModel> Handle(RequestModel request, CancellationToken ct)
    {
        // Implementation
    }
}
```

### Endpoint Groups

Groups provide shared configuration for related endpoints:

```csharp
public class UserEndpointGroup : IEndpointGroup
{
    public static RouteGroupBuilder Configure(WebApplication app)
    {
        return app.MapGroup("/api/v1/users")
                  .WithTags("Users")
                  .RequireAuthorization();
    }
}

// Usage in endpoints
public static void Configure(RouteHandlerBuilder builder)
{
    builder.Group<UserEndpointGroup>()
           .Get("/{id}");
}
```

### Generated Extension Methods

The source generator creates two main extension methods:

1. **`AddEndpointsFrom{AssemblyName}()`** - Registers endpoint services with DI container
2. **`MapEndpointsFrom{AssemblyName}()`** - Maps all endpoints to the application's route table

## Integration Testing Architecture

### Test Infrastructure Components

#### **ExampleApiWebApplicationFactory**
Custom `WebApplicationFactory<Program>` that:
- Overrides `ConfigureWebHost()` for test-specific service registration
- Replaces production services with test doubles (e.g., `ITodoStore` → `TestTodoStore`)
- Configures "Testing" environment
- Ensures isolated test execution

#### **TestTodoStore** 
In-memory implementation providing:
- Full CRUD operations via `Dictionary<int, Todo>`
- Test lifecycle methods: `Clear()`, `SeedData(params Todo[] todos)`
- Deterministic ID generation and timestamp handling
- Predictable state management between tests

#### **TestHelpers**
Static utility class with:
- `CreateJsonContent<T>()` - JSON serialization with custom options

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IeuanWalker/MinimalApi.Endpoints](https://github.com/IeuanWalker/MinimalApi.Endpoints) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
