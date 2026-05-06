---
trigger: always_on
description: Cheat Engine MCP Server — a C# plugin that exposes Cheat Engine functionality as MCP tools over Streamable HTTP using the official [Model Context Protocol C# SDK](https://github.com/modelcontextprotocol/csharp-sdk).
---

# Project Guidelines

## Project Overview

Cheat Engine MCP Server — a C# plugin that exposes Cheat Engine functionality as MCP tools over Streamable HTTP using the official [Model Context Protocol C# SDK](https://github.com/modelcontextprotocol/csharp-sdk).

**Key Architecture**:
- **MCP Server**: ASP.NET Core app on `http://127.0.0.1:6300` with Streamable HTTP at `/`
- **CE Plugin Host**: Runs inside Cheat Engine process, manages server lifecycle via menu
- **CESDK Submodule**: C# wrapper for CE's Lua API (git submodule dependency)
- **Single DLL**: All dependencies embedded, copy to CE plugins folder to deploy

## Cheat Engine Lua API Reference

The full CE Lua API documentation is at `C:\Program Files\Cheat Engine\celua.txt`. Always consult this file when:
- Adding new CESDK wrapper methods or tools
- Verifying correct Lua function names, parameters, and return types
- Understanding CE object models (MemScan, FoundList, AddressList, MemoryRecord, etc.)

## Build and Test

```bash
# Initialize submodule (first time only)
git submodule update --init --recursive

# Build Debug
dotnet build

# Build Release
dotnet build -c Release
```

**Deploy**: Copy `ce-mcp.dll` from `bin/x64/Debug/net10.0-windows/` (or Release) to Cheat Engine plugins directory, enable in CE, use "MCP" → "Start MCP Server" menu.

**Requirements**: .NET 10.0 SDK, CE 7.6.2+, ASP.NET Core 10.0 runtime. CE must have `ce.runtimeconfig.json` set to .NET 10.0 (see README).

## Architecture

### Core Components

- **Plugin.cs** — Main CE plugin entry point (`McpPlugin : CheatEnginePlugin`). Manages MCP server lifecycle, registers Lua functions for CE menu integration (`toggle_mcp_server`, `show_mcp_config`), provides WPF config UI. Handles assembly resolution for WPF components.
- **McpServer.cs** — MCP server using `ModelContextProtocol.AspNetCore`. Registers all tools via `WithTools<T>()`, resources via `WithResources<T>()`, and maps endpoints with `MapMcp()`. Runs ASP.NET Core with minimal logging in background task.
- **ServerConfig.cs** — Configuration management (host/port/name). Loads from `%APPDATA%\CeMCP\config.json` with env var overrides (`MCP_HOST`, `MCP_PORT`). Uses source-generated JSON serialization.
- **ThemeHelper.cs** — Cross-platform dark mode detection (Windows registry, macOS `defaults`, Linux GTK settings)

### Tools (`src/Tools/`)

All tools use `[McpServerToolType]` on the class and `[McpServerTool(Name = "tool_name")]` + `[Description]` on methods. Tools are `public class` with a `private Constructor() {}` and static methods (not `static class` — required for MCP SDK DI). Each returns anonymous objects with `success` boolean and either result data or `error` message.

- **ProcessTool** — Process and thread management (list/open processes, get current process ID)
- **MemoryTool** — Memory read and write (bytes, int8/16/32/64, float, double, string, AOB patterns)
- **ScanTool** — Memory scanning (first scan, next scan, reset, AOB scan) using MemScan/FoundList
- **AssemblyTool** — Disassembly and address resolution (symbol lookups)
- **AutoAssemblyTool** — Single-instruction assembly, Auto Assembler script execution and syntax checking
- **MemoryViewTool** — Memory view operations: disassemble ranges, navigate code backwards, enumerate memory regions, query protections, set comments
- **SymbolTool** — Symbol/module management: enumerate loaded modules, symbol lookup, enable Windows/kernel symbols, pointer size management
- **AddressListTool** — Cheat table CRUD operations (get/add/update/delete/clear records)
- **LuaExecutionTool** — Execute arbitrary Lua scripts in CE with stack management
- **ConversionTool** — String format conversion (MD5, ANSI/UTF8)

### Resources (`src/Resources/`)

> **Not yet implemented** — `src/Resources/` does not exist yet. Resources use `[McpServerResourceType]` on class and `[McpServerResource(UriTemplate = "scheme://path")]` + `[Description]` on methods. Resources are read-only state/data (vs tools which perform actions). Return JSON strings.

### SDK Layer (`CESDK/`)

Git submodule — C# wrapper around Cheat Engine's Lua API. Key classes: `MemoryAccess`, `Process`, `AOBScanner`, `Assembler`, `Disassembler`, `AddressResolver`, `MemScan`, `FoundList`, `AddressList`, `ThreadList`, `Converter`, `Speedhack`, `Debugger`, `SymbolWaiter`, `SymbolManager`, `MemoryRegions`.

### Views (`src/Views/`)

- **ConfigWindow.xaml/.cs** — WPF config window. Supports dark/light theme via `ThemeHelper`. Runs in STA thread with dispatcher for cross-thread UI updates.

## Adding New Tools

1. Create a new file in `src/Tools/` with `[McpServerToolType]` class attribute
2. Add static methods with `[McpServerTool(Name = "tool_name")]` and `[Description("...")]`
3. Use `[Description]` on parameters for MCP schema generation
4. Return anonymous objects: `new { success = true, ... }` or `new { success = false, error = "..." }`
5. Register in `McpServer.cs` via `.WithTools<Tools.YourTool>()`
6. Consult `C:\Program Files\Cheat Engine\celua.txt` for correct Lua function signatures

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShadowNineX/ce-mcp](https://github.com/ShadowNineX/ce-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
