---
trigger: always_on
description: This guide provides agentic coding agents with the essential information to work effectively in the yes-ue-mcp codebase.
---

# AGENTS.md - Development Guide for yes-ue-mcp

This guide provides agentic coding agents with the essential information to work effectively in the yes-ue-mcp codebase.

## Project Overview

**yes-ue-mcp** is a native C++ Unreal Engine plugin implementing the Model Context Protocol (MCP) over HTTP. It enables AI assistants to inspect, analyze, and modify UE projects through a standardized JSON-RPC API.

- **Type**: Unreal Engine 5.6+ C++ plugin
- **Architecture**: Two-module system (Runtime + Editor)
- **Protocol**: MCP 2025-03-26 (Streamable HTTP) with JSON-RPC 2.0
- **Transport**: HTTP server on localhost:8080/mcp

## Build System & Commands

### Build Commands
```bash
# Build plugin with UnrealBuildTool
<UE>/Engine/Build/BatchFiles/Build.bat YourProjectEditor Win64 Development

# Regenerate project files after adding files
<UE>/Engine/Binaries/DotNET/UnrealBuildTool/UnrealBuildTool.exe -projectfiles -project="YourProjectPath/YourProject.uproject" -game -rocket -progress
```

### Test Commands
```bash
# Run Python integration tests (requires UE Editor running with plugin)
cd Tests
python -m pytest test_mcp_tools.py -v
python -m unittest test_mcp_tools -v
python test_mcp_tools.py

# Run single test
python -m pytest test_mcp_tools.py::TestConnection::test_server_responds -v
```

### Environment Variables for Testing
- `MCP_HOST`: MCP server host (default: 127.0.0.1)
- `MCP_PORT`: MCP server port (default: 8080)

### Development Workflow
```bash
# Deploy plugin to test projects
.\copy_plugin.ps1                    # Copy to both projects
.\copy_plugin.ps1 -Target Elpis      # Copy to Elpis only
.\copy_plugin.ps1 -Target GameAnim   # Copy to GameAnimationSample56 only
```

## Code Style Guidelines

### General Conventions
- Follow Epic Games C++ coding standards
- Use `YESUEMCP_API` / `YESUEMCPEDITOR_API` for exported symbols
- Copyright header: `// Copyright softdaddy-o 2024. All Rights Reserved.`
- Use `TEXT()` macro for string literals
- Use `TEXT()` for log messages and UE API calls

### File Organization
```
Source/
├── YesUeMcp/                    # Runtime module
│   ├── Public/                  # Public headers
│   │   ├── Protocol/           # MCP protocol types
│   │   └── Tools/              # Base tool classes
│   └── Private/                # Implementation files
└── YesUeMcpEditor/             # Editor module
    ├── Public/                  # Public headers
    │   ├── Server/             # HTTP server
    │   ├── Subsystem/          # Editor subsystems
    │   ├── Tools/              # Tool implementations
    │   │   ├── Blueprint/      # Blueprint tools
    │   │   ├── Asset/          # Asset tools
    │   │   ├── Level/          # Level tools
    │   │   ├── Write/          # Write operations
    │   │   └── ...
    │   └── Utils/              # Utility classes
    └── Private/                # Implementation files
```

### Naming Conventions
- **Classes**: `U` prefix for UObject classes, `F` for structs, `E` for enums
- **Files**: Match class name (e.g., `QueryBlueprintTool.h` for `UQueryBlueprintTool`)
- **Functions**: PascalCase (e.g., `GetToolName()`, `ExecuteTool()`)
- **Variables**: camelCase (e.g., `toolName`, `assetPath`)
- **Constants**: UPPER_CASE (e.g., `YESUEMCP_VERSION`)

### Tool Development Pattern
```cpp
// Header (.h)
UCLASS()
class YESUEMCPEDITOR_API UMyTool : public UMcpToolBase
{
    GENERATED_BODY()

public:
    virtual FString GetToolName() const override { return TEXT("my-tool"); }
    virtual FString GetToolDescription() const override;
    virtual TMap<FString, FMcpSchemaProperty> GetInputSchema() const override;
    virtual TArray<FString> GetRequiredParams() const override;
    virtual FMcpToolResult Execute(const TSharedPtr<FJsonObject>& Arguments, const FMcpToolContext& Context) override;
};

// Implementation (.cpp)
REGISTER_MCP_TOOL(UMyTool)  // Auto-registration

FString UMyTool::GetToolDescription() const
{
    return TEXT("Tool description for tools/list");
}

// ... other methods
```

### Error Handling
- Use `FMcpToolResult::Error()` for tool failures
- Always validate required parameters before processing
- Use `UE_LOG(LogYesUeMcp, Warning, TEXT("..."))` for warnings
- Return structured error messages with context

### Memory Management
- Use `TSharedPtr` for JSON objects and smart pointers
- Follow UE object ownership rules (UObjects are garbage collected)
- Use `FScopedTransaction` for write operations to support undo/redo

## Module Architecture

### YesUeMcp (Runtime Module)
- **Purpose**: Core MCP protocol layer
- **Dependencies**: Core, CoreUObject, Json, JsonUtilities
- **Key Classes**:
  - `UMcpToolBase`: Abstract base class for all tools
  - `FMcpToolRegistry`: Tool registration and discovery
  - `FMcpTypes`: MCP protocol type definitions

### YesUeMcpEditor (Editor Module)
- **Purpose**: HTTP server + UE Editor tool implementations
- **Dependencies**: All YesUeMcp deps + HTTP, UnrealEd, Kismet, etc.
- **Key Classes**:
  - `FMcpServer`: HTTP server handling MCP requests
  - `UMcpEditorSubsystem`: Lifecycle management
  - Tool implementations in `Tools/` subdirectories

### Tool Categories
- **Read Tools (10)**: Query and analyze UE assets

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [softdaddy-o/yes-ue-mcp](https://github.com/softdaddy-o/yes-ue-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
