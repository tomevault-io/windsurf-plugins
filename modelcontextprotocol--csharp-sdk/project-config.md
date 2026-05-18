---
trigger: always_on
description: This repository contains the official C# SDK for the Model Context Protocol (MCP), enabling .NET applications to implement and interact with MCP clients and servers.
---

# Copilot Instructions for MCP C# SDK

This repository contains the official C# SDK for the Model Context Protocol (MCP), enabling .NET applications to implement and interact with MCP clients and servers.

## Critical: Always Build and Test

**ALWAYS build and run tests before declaring any task complete or making a pull request.**

When making code changes:
1. **Build first**: Run `dotnet build` to ensure the code compiles without errors
2. **Run tests**: Execute `dotnet test` to verify all tests pass
3. **Fix issues**: Address any build errors or test failures before proceeding
4. **Verify iteratively**: Build and test frequently during development, not just at the end
5. **Check warnings**: Treat warnings as errors - the build is configured with `TreatWarningsAsErrors=true`

**Never skip these steps.** Even small changes can have unexpected impacts. A passing build and test suite is the minimum bar for any code change.

## Project Overview

The SDK consists of three main packages:
- **ModelContextProtocol.Core** - Client and low-level server APIs with minimal dependencies
- **ModelContextProtocol** - The main package with hosting and dependency injection extensions and which references ModelContextProtocol.Core
- **ModelContextProtocol.AspNetCore** - HTTP-based MCP server implementations for ASP.NET Core, referencing ModelContextProtocol

## C# Coding Standards

### Language Features
- Use **file-scoped namespaces** for all C# files
- Enable **implicit usings** and **nullable reference types**
- Use **preview language features** (LangVersion: preview)
- Treat warnings as errors

### Code Style
- Follow the conventions in `.editorconfig`
- Use clear, descriptive XML documentation comments for public APIs
- Follow async/await patterns consistently
- Use file-scoped namespaces: `namespace ModelContextProtocol.Client;`

### Naming Conventions
- Use `McpClient`, `McpServer`, `McpSession` for MCP-related classes (capitalize MCP)
- Prefix MCP-specific types with `Mcp` (e.g., `McpException`, `McpEndpoint`)
- Use descriptive names for parameters with `[Description("...")]` attributes when exposing to MCP

## Architecture Patterns

### Dependency Injection
- Use Microsoft.Extensions.DependencyInjection patterns
- Register services with `.AddMcpServer()` and `.AddMcpClient()` extension methods
- Support both builder patterns and options configuration

### JSON Serialization
- Use `System.Text.Json` exclusively for all JSON operations
- Use `McpJsonUtilities.DefaultOptions` for consistent serialization settings across the SDK
- Support source generation for Native AOT compatibility via `McpJsonUtilities` source generators
- Set `JsonIgnoreCondition.WhenWritingNull` for optional properties to minimize payload size
- Use `JsonSerializerDefaults.Web` for camelCase property naming
- Protocol types are decorated with `[JsonSerializable]` attributes for AOT support
- Custom converters: `CustomizableJsonStringEnumConverter` for flexible enum serialization

### Async Patterns
- All I/O operations should be async
- Use `ValueTask<T>` for hot paths that may complete synchronously
- Always accept `CancellationToken` parameters for async operations
- Name parameters consistently: `cancellationToken`

### MCP Protocol
- Follow the MCP specification at https://spec.modelcontextprotocol.io/ ([specification docs](https://github.com/modelcontextprotocol/modelcontextprotocol/tree/main/docs/specification))
- Use JSON-RPC 2.0 for message transport
- Support all standard MCP capabilities (e.g. tools, prompts, resources, sampling)
- Implement proper error handling with `McpException` and `McpErrorCode`

### Error Handling
- Throw `McpException` for MCP protocol-level errors with appropriate `McpErrorCode`
- Use standard error codes: `InvalidRequest`, `MethodNotFound`, `InvalidParams`, `InternalError`
- Let domain exceptions bubble up and convert to `InternalError` at transport boundary
- Include detailed error messages in exception `Message` property for debugging
- Errors are automatically converted to JSON-RPC error responses by the server infrastructure

## Testing

### Test Organization
- Unit tests in `tests/ModelContextProtocol.Tests` for core functionality
- Integration tests in `tests/ModelContextProtocol.AspNetCore.Tests` for HTTP/SSE transports
- Shared test utilities in `tests/Common/Utils/`
- Test servers in `tests/ModelContextProtocol.Test*Server/` for integration scenarios
- Filter manual tests with `[Trait("Execution", "Manual")]` - these require external dependencies

### Test Base Classes
- **`LoggedTest`**: Base class that wires up `ILoggerFactory` with `XunitLoggerProvider` (test output) and `MockLoggerProvider` (log assertions). Inherit from this for any test needing logging.
- **`ClientServerTestBase`**: Sets up in-memory client/server pair via `Pipe`. Override `ConfigureServices` to register tools/prompts/resources, then call `CreateMcpClientForServer()`. Handles async disposal automatically.
- **`KestrelInMemoryTest`** (AspNetCore tests): Hosts ASP.NET Core with in-memory transport — no ports needed.
- **`TestServerTransport`**: In-memory mock transport for testing client logic without a real server.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [modelcontextprotocol/csharp-sdk](https://github.com/modelcontextprotocol/csharp-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
