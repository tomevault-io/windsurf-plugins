---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

**MCP for Unity** is a bridge that lets AI assistants (Claude, Cursor, Windsurf, etc.) control the Unity Editor through the Model Context Protocol (MCP). It enables AI-driven game development workflows - creating GameObjects, editing scripts, managing assets, running tests, and more.

## Architecture

```text
AI Assistant (Claude/Cursor)
        ↓ MCP Protocol (stdio/HTTP)
Python Server (Server/src/)
        ↓ WebSocket + HTTP
Unity Editor Plugin (MCPForUnity/)
        ↓ Unity Editor API
Scene, Assets, Scripts
```

**Two codebases, one system:**
- `Server/` - Python MCP server using FastMCP
- `MCPForUnity/` - Unity C# Editor package

### Three Layers on the Python Side

The Python server has three distinct layers. These are **not** auto-generated from each other:

| Layer | Location | Framework | Purpose |
|-------|----------|-----------|---------|
| **MCP Tools** | `Server/src/services/tools/` | FastMCP (`@mcp_for_unity_tool`) | Exposed to AI assistants via MCP protocol |
| **CLI Commands** | `Server/src/cli/commands/` | Click (`@click.command`) | Terminal interface for developers |
| **Resources** | `Server/src/services/resources/` | FastMCP (`@mcp_for_unity_resource`) | Read-only state exposed to AI assistants |

MCP tools call Unity via WebSocket (`send_with_unity_instance`). CLI commands call Unity via HTTP (`run_command`). Both route to the same C# `HandleCommand` methods.

### Transport Modes

- **Stdio**: Single-agent only. Separate Python process per client. Legacy TCP bridge to Unity. New connections stomp old ones.
- **HTTP**: Multi-agent ready. Single shared Python server. WebSocket hub at `/hub/plugin`. Session isolation via `client_id`.

## Code Philosophy

### 1. Domain Symmetry
Python MCP tools mirror C# Editor tools. Each domain exists in both:
- `Server/src/services/tools/manage_material.py` ↔ `MCPForUnity/Editor/Tools/ManageMaterial.cs`
- CLI commands (`Server/src/cli/commands/`) also mirror these but are a separate implementation.

### 2. Minimal Abstraction
Avoid premature abstraction. Three similar lines of code is better than a helper that's used once. Only abstract when you have 3+ genuine use cases.

### 3. Delete Rather Than Deprecate
When removing functionality, delete it completely. No `_unused` renames, no `// removed` comments, no backwards-compatibility shims for internal code.

### 4. Test Coverage Required
Every new feature needs tests. Run them before PRs.

### 5. Keep Tools Focused
Each MCP tool does one thing well. Resist the urge to add "convenient" parameters that bloat the API surface.

### 6. Use Resources for Reading
Keep them smart and focused rather than "read everything" type resources. Resources should be quick and LLM-friendly.

## Key Patterns

### Python MCP Tool Registration
Tools in `Server/src/services/tools/` are auto-discovered. Use the `@mcp_for_unity_tool` decorator:
```python
from services.registry import mcp_for_unity_tool

@mcp_for_unity_tool(
    description="Does something in Unity.",
    group="core",  # core (default), vfx, animation, ui, scripting_ext, testing, probuilder, profiling, docs
)
async def manage_something(
    ctx: Context,
    action: Annotated[Literal["create", "delete"], "Action to perform"],
) -> dict[str, Any]:
    unity_instance = await get_unity_instance_from_context(ctx)
    params = {"action": action}
    response = await send_with_unity_instance(async_send_command_with_retry, unity_instance, "manage_something", params)
    return response
```

The `group` parameter controls tool visibility. Only `"core"` is enabled by default. Non-core groups (vfx, animation, etc.) start disabled and are toggled via `manage_tools`.

### Python CLI Error Handling
CLI commands (not MCP tools) use the `@handle_unity_errors` decorator:
```python
@handle_unity_errors
async def my_command(ctx, ...):
    result = await call_unity_tool(...)
```

### C# Tool Registration
Tools are auto-discovered by `CommandRegistry` via reflection. Use the `[McpForUnityTool]` attribute:
```csharp
[McpForUnityTool("manage_something", AutoRegister = false, Group = "core")]
public static class ManageSomething
{
    // Sync handler (most tools):
    public static object HandleCommand(JObject @params)
    {
        var p = new ToolParams(@params);
        // ...
        return new SuccessResponse("Done.", new { data = result });
    }

    // OR async handler (for long-running operations like play-test, refresh, batch):
    public static async Task<object> HandleCommand(JObject @params)
    {
        // CommandRegistry detects Task return type automatically
        await SomeAsyncOperation();
        return new SuccessResponse("Done.");
    }
}
```

Async handlers use `EditorApplication.update` polling with `TaskCompletionSource` — see `RefreshUnity.cs` for the canonical pattern.

### C# Parameter Handling
Use `ToolParams` for consistent parameter validation:
```csharp
var p = new ToolParams(parameters);
var pageSize = p.GetInt("page_size", "pageSize") ?? 50;
var name = p.RequireString("name");
```

### C# Resources

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CoplayDev/unity-mcp](https://github.com/CoplayDev/unity-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
