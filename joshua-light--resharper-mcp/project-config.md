---
trigger: always_on
description: An MCP (Model Context Protocol) server that runs **inside** the ReSharper/Rider backend process,
---

# ReSharper MCP — Project Guide

## What This Is

An MCP (Model Context Protocol) server that runs **inside** the ReSharper/Rider backend process,
exposing code intelligence features to AI assistants via HTTP.

## Architecture

### Two-part Rider plugin

Rider plugins require both a JVM frontend and a .NET backend:

- `rider-plugin/META-INF/plugin.xml` — minimal IntelliJ plugin descriptor (no Java/Kotlin code needed for backend-only plugins). Without this JAR, Rider silently ignores the `dotnet/` folder.
- `src/ReSharperMcp/` — the .NET backend that runs inside the ReSharper host process.

### Installed plugin layout (in `~/.local/share/JetBrains/Rider2025.3/ReSharperMcp/`)

```
lib/ReSharperMcp.jar       # plugin.xml descriptor (tells Rider this plugin exists)
dotnet/ReSharperMcp.dll    # backend component (loaded into ReSharper host)
```

### MCP transport

The official C# MCP SDK (`ModelContextProtocol` NuGet) requires .NET 8+ for HTTP transport,
but ReSharper plugins target **net472**. So we implement a lightweight MCP server manually:

- `McpHttpServer.cs` — `HttpListener`-based HTTP server on `127.0.0.1:23741`
- Implements JSON-RPC 2.0 directly (handles `initialize`, `tools/list`, `tools/call`)
- Protocol types in `Protocol/JsonRpc.cs` and `Protocol/McpTypes.cs`
- Uses `Newtonsoft.Json` (bundled with Rider's ReSharper host, no need to ship it)

### Multi-solution support (primary/peer architecture)

Rider opens each solution in a separate OS process, so multiple instances compete for ports.
The plugin uses a **primary/peer** model:

- **Primary** — the first Rider instance binds the base port (23741). It acts as the single MCP endpoint for all clients.
- **Peer** — subsequent instances bind the next available port (23742, 23743, ...) and register themselves with the primary via `internal/register` / `internal/deregister` JSON-RPC methods.
- The primary proxies `tools/call` requests for peer solutions by forwarding HTTP requests to the peer's port. If a proxy fails (peer crashed), the stale registration is automatically removed.

Component breakdown:

- `McpShellComponent.cs` — `[ShellComponent]` (process-level singleton). Tries to bind the base port; on failure, increments and becomes a peer. Owns the `McpHttpServer`. Handles peer registration notifications.
- `McpServerComponent.cs` — `[SolutionComponent]` (per-solution). Registers/unregisters tools with the shell component on solution open/close. Handles PSI thread dispatch.
- When only one solution is open (across all Rider instances), all tool calls route to it automatically (backwards-compatible).
- When multiple solutions are open, callers must specify `solutionName` in tool arguments.
- The `list_solutions` meta-tool returns all currently open solutions (local + peers).
- `solutionName` matches case-insensitively against the solution filename (without `.sln`) or the full path.

### ReSharper integration

- `PsiHelpers.cs` — shared helpers: file lookup, position-to-node resolution, element resolution, snippet truncation.
- `Tools/IMcpTool.cs` — common interface for all tools. Each tool provides its name, description, JSON schema, and execute method.
- `Tools/IMcpWriteTool.cs` — marker interface for tools that modify the PSI tree (requires write lock + PsiTransaction).

## Tools

| Tool | Description |
|------|-------------|
| `find_usages` | Find all references to a symbol (uses `IFinder.FindReferences()`) |
| `get_symbol_info` | Get detailed symbol info: kind, type, params, docs, base types, declaration location |
| `find_implementations` | Find implementations of interfaces/abstract classes and overrides of virtual members |
| `get_file_errors` | Get compile errors and unresolved references by walking the PSI tree |
| `search_symbol` | Search symbols by name (substring match) across the solution |
| `go_to_definition` | Navigate from a usage site or symbol name to the declaration location |
| `get_solution_structure` | List all projects, target frameworks, and project-to-project references |
| `browse_namespace` | Browse namespace hierarchy: child namespaces and types in a namespace |
| `list_symbols_in_file` | List all declarations in a file (types, methods, properties, etc.) |
| `list_solutions` | List all currently open solutions (server-level meta-tool) |
| `fix_usings` | Fix missing C# using directives by resolving unresolved type references against the symbol cache |
| `flow` | Describe control flow of a method or type: execution steps, branches, loops, error paths, inlined call targets, why-hints |

### Symbol resolution

All tools that take a symbol accept **two modes**:
- `filePath` + `line` + `column` (1-based) — resolve from a specific code position
- `symbolName` — resolve by name (e.g. `"MyClass"`, `"Namespace.MyClass"`, `"MyClass.MyMethod"`)
- Optional `kind` filter (`"type"`, `"method"`, `"property"`, `"field"`, `"event"`) to disambiguate

When multiple symbols match a name, tools return an **ambiguity error** listing all candidates with their qualified names, kinds, and locations — so the caller can pick the right one.

`search_symbol`, `get_file_errors`, `get_solution_structure`, `browse_namespace`, and `list_symbols_in_file` use their own input formats.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joshua-light/resharper-mcp](https://github.com/joshua-light/resharper-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
