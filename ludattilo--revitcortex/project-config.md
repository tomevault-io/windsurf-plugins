---
trigger: always_on
description: Flussi operativi collaudati: vedere WORKFLOWS.md
---

# RevitCortex -- AI Assistant Guide

Flussi operativi collaudati: vedere WORKFLOWS.md

## Project Overview

RevitCortex is a next-generation MCP (Model Context Protocol) server for Autodesk Revit. It improves on the original mcp-servers-for-revit with typed errors, session state, and dynamic tool discovery. Tools are rewritten from scratch -- not copied from the fork reference.

## Supported Revit Versions

| Revit Version | Target Framework |
|---------------|-----------------|
| 2023          | net48           |
| 2024          | net48           |
| 2025          | net8.0-windows  |
| 2026          | net8.0-windows  |
| 2027          | net10.0-windows |

## Cross-Target Compatibility (net48 vs net8+)

R2023 and R2024 target **net48**. Code that compiles on net8+ may fail on net48. Before using any C# feature, check this list:

| Feature | net8+ | net48 | Fix |
|---------|-------|-------|-----|
| `record` types | OK | **ERROR** CS0518 (`IsExternalInit` missing) | Use `class` with readonly properties and constructor |
| `Dictionary.GetValueOrDefault()` | OK | **ERROR** CS1061 | Use `TryGetValue` with ternary |
| `init` property accessors | OK | **ERROR** CS0518 | Use `{ get; }` + constructor |
| `Index`/`Range` (`^1`, `..`) | OK | **ERROR** | Use `.Length - 1`, `.Substring()` |
| `IAsyncEnumerable<T>` | OK | **ERROR** | Not available on net48 |
| `file`-scoped types | OK | **ERROR** | Use `internal` |
| Default interface methods | OK | **ERROR** | Move to abstract class or separate method |

**Rule**: After adding/modifying any C# file, always build for BOTH `Debug R25` (net8) AND `Debug R24` (net48) before committing. A green R25 build does NOT guarantee R24 will compile.

## Project Structure

```
RevitCortex/
  RevitCortex.sln
  nuget.config
  src/
    RevitCortex.Core/         Core types (no Revit dependency)
      Discovery/                DocumentCapabilities, IDocumentAnalyzer
      Results/                  CortexResult<T>, CortexError, CortexErrorCode
      Session/                  CortexSession, ISessionStore, SessionStore
      Tools/                    ICortexTool interface
    RevitCortex.Plugin/       Revit add-in (ExternalApplication)
      Communication/            SocketService, JsonRpcModels
      Discovery/                DocumentAnalyzer, LocaleDetector
      CortexRouter.cs           Request dispatch
      RevitCortexApp.cs         Entry point
    RevitCortex.Tools/        Tool implementations
      Meta/                     SayHelloTool
    RevitCortex.Tests/        Unit tests (xUnit)
      Discovery/                DocumentCapabilitiesTests
      Results/                  CortexResultTests
      Router/                   CortexRouterTests, FakeTool, FakeAnalyzer
      Session/                  SessionStoreTests
  src/RevitCortex.Server/     C# MCP server (stdio transport)
    Program.cs                  Entry point (MCP hosting)
    Connection/RevitBridge.cs   TCP bridge to Plugin
    Tools/                      157 tool definitions (9 files)
```

## Architecture: Layer Cake

```
MCP Server (C#) -> SocketService -> CortexRouter -> ICortexTool
                                                |
                                          CortexSession
                                                |
                                    DocumentCapabilities
```

- **MCP Server (C#)** -- ModelContextProtocol SDK, stdio transport to Codex. Located in `src/RevitCortex.Server/`.
- **SocketService (C#)** -- TCP listener, JSON-RPC framing between C# MCP server and C# plugin.
- **CortexRouter (C#)** -- Deserializes the JSON-RPC request, finds the matching ICortexTool by name, invokes it with CortexSession, returns CortexResult.
- **ICortexTool (C#)** -- Unified interface for all tools.
- **CortexSession (C#)** -- Shared state facade passed to every tool: session store, document capabilities, detected locale.
- **CortexResult\<T\> (C#)** -- Typed response envelope with success/error discriminator and structured error codes.
- **DocumentAnalyzer (C#)** -- Scans the active Revit document to populate DocumentCapabilities, enabling/disabling dynamic tools.

## Build Commands

### C# (from repo root)

```bash
dotnet build -c "Debug R25" src/RevitCortex.Plugin/RevitCortex.Plugin.csproj   # Revit 2025
dotnet build -c "Debug R24" src/RevitCortex.Plugin/RevitCortex.Plugin.csproj   # Revit 2024
dotnet build -c "Debug R23" src/RevitCortex.Plugin/RevitCortex.Plugin.csproj   # Revit 2023
dotnet build -c "Debug R26" src/RevitCortex.Plugin/RevitCortex.Plugin.csproj   # Revit 2026
dotnet build -c "Debug R27" src/RevitCortex.Plugin/RevitCortex.Plugin.csproj   # Revit 2027
```

### C# MCP Server

```bash
dotnet build src/RevitCortex.Server/RevitCortex.Server.csproj
```

### Tests

```bash
dotnet test src/RevitCortex.Tests/RevitCortex.Tests.csproj -c "Debug R26"
```

> **Note:** Run tests by specifying the test project directly, not from the solution root.
> `dotnet test -c "Debug R26"` from the root fails because NuGet restore cannot resolve
> the `$(RevitVersion).*` package version wildcard used in Plugin and Tools projects
> without a per-project build context. The test project itself has no such dependency.

## Key Patterns

### CortexResult\<T\>


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LuDattilo/RevitCortex](https://github.com/LuDattilo/RevitCortex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
