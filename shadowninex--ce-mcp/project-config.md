---
trigger: always_on
description: `ce-mcp` builds `ce-mcp.dll`, an x64 Cheat Engine 7.6.2+ plugin that hosts a stateless MCP server over Streamable HTTP. It exposes process control, memory/pointer/scan, assembly/analysis, symbol/RTTI, Structure Dissect, cheat-table, injection, debugger, optional DBVM, address-list, conversion, and Lua operations. The plugin targets `net10.0-windows`, uses WPF for configuration, and embeds managed dependencies into one DLL.
---

# Repository Guidelines

## Project Overview

`ce-mcp` builds `ce-mcp.dll`, an x64 Cheat Engine 7.6.2+ plugin that hosts a stateless MCP server over Streamable HTTP. It exposes process control, memory/pointer/scan, assembly/analysis, symbol/RTTI, Structure Dissect, cheat-table, injection, debugger, optional DBVM, address-list, conversion, and Lua operations. The plugin targets `net10.0-windows`, uses WPF for configuration, and embeds managed dependencies into one DLL.

## Architecture & Data Flow

1. Cheat Engine loads the plugin through `CESDK`, which initializes shared Lua/native state and calls `McpPlugin.OnEnable`.
2. `src/Plugin.cs` installs the `MCP` menu, loads `%APPDATA%\CeMCP\config.json`, applies `MCP_HOST`/`MCP_PORT` overrides, and starts the WPF configuration UI or server.
3. `src/McpServer.cs` builds the ASP.NET Core host, registers tool classes, and maps the stateless Streamable HTTP endpoint.
4. A method in `src/Tools/` validates an MCP request, marshals Cheat Engine work to the main GUI thread, and calls a typed `CESDK` facade.
5. `CESDK/src/Classes/` calls the CE Lua API through `LuaUtils`/`LuaNative`; tools return JSON-visible `{ success = true, ... }` or `{ success = false, error }` objects.

Dependency direction is `MCP tool -> CESDK typed facade -> LuaUtils/LuaNative -> Cheat Engine`. ASP.NET Core may handle concurrent requests, but CE Lua state and engine objects are not thread-safe: serialize CE-facing work with `ToolThread.OnMainThread(...)`. Keep process-attached checks and the subsequent operation in the same main-thread block.

State is deliberately centralized: static `ServerConfig`, shared `PluginContext.Lua`, server fields on `McpPlugin`/`McpServer`, and named scanner state in `ScanTool`. Scanner/found-list lifecycles are order-sensitive: deinitialize old results, run the scan, call `WaitTillDone()`, initialize results, then read them.

## Key Directories

- `src/`: plugin lifecycle, MCP host, schema transforms, configuration, and WPF application code.
- `src/Tools/`: client-facing MCP tool adapters grouped by CE capability.
- `src/Models/`, `src/Views/`: WPF configuration state and UI.
- `CESDK/src/`: submodule-provided native plugin bootstrap, Lua interop, and typed CE wrappers compiled into the plugin.
- `tests/CeMCP.Tests/Unit/`: deterministic tests that do not require Cheat Engine.
- `tests/CeMCP.Tests/Live/`: opt-in tests against a running CE-hosted MCP server.
- `CESDK/tests/`: separate CE-loaded live-test plugin and report-validating MSTest host.
- `skills/ce-mcp/`: distributable AI skill that must track the public MCP surface.
- `.github/workflows/`: Windows build/artifact and SonarCloud pipelines.

## Development Commands

Run from the repository root in PowerShell:

```powershell
git submodule update --init --recursive
dotnet restore
dotnet build
dotnet test --filter "TestCategory!=Live"
dotnet build -c Release
```

CI-equivalent build sequence:

```powershell
dotnet restore
dotnet build -c Debug --no-restore
dotnet test -c Debug --no-restore --no-build --filter "TestCategory!=Live"
dotnet build -c Release --no-restore
```

Debug output is `bin/x64/Debug/net10.0-windows/ce-mcp.dll`; Release uses the corresponding `Release` directory. There is no repository lint or formatter command. SonarCloud is the configured static-analysis gate.

Manual run: copy the built DLL to Cheat Engine's plugins directory, restart CE, enable the plugin, choose `MCP` -> start server, and connect to `http://localhost:6300/` by default. This is a plugin, not a standalone application.

## Code Conventions & Common Patterns

- Use C# with 4-space indentation, nullable reference types, warnings as errors, and the existing brace style. Add XML summaries to public wrapper APIs; keep comments focused on CE/Lua edge cases.
- MCP tool containers are `public class` types with `[McpServerToolType]` and a private constructor. Tool methods are synchronous static `object` methods decorated with `[McpServerTool(Name = "snake_case")]`; describe every public parameter.
- Register every tool class explicitly in `McpServer.Start` with `WithToolsAndSchemaTransform<T>()`. Never use plain `WithTools<T>()`: `SchemaTransform` removes nullable type arrays and oversized numeric schema keywords that break some MCP clients.
- Validate required inputs before native calls. Return anonymous structured objects with `success` and result fields, or `success = false` plus `error`. Catch operational exceptions at the tool boundary; CESDK wrappers translate lower-level failures to domain-specific exceptions.
- Use `ToolThread.OnMainThread(...)` for new CE-facing tool bodies. Follow a subsystem's established pattern when modifying older scan, address-list, Lua, or debugger code; do not casually move CE work onto HTTP worker threads.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShadowNineX/ce-mcp](https://github.com/ShadowNineX/ce-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
