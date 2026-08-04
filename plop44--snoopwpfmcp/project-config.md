---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

### Main Solution
```bash
# Build the entire solution
dotnet build SnoopMcp.sln

# Build in Release mode
dotnet build SnoopMcp.sln --configuration Release

# Clean and rebuild
dotnet clean SnoopMcp.sln
dotnet build SnoopMcp.sln
```

### Individual Projects
```bash
# Build MCP Server (main entry point)
dotnet build MCP/Injector/SnoopWpfMcpServer.csproj

# Build WpfInspector library
dotnet build MCP/WpfInspector/WpfInspector.csproj

# Build TestApp for testing
dotnet build MCP/TestApp/TestApp.csproj
```

### Running the MCP Server
```bash
# Run as MCP server (primary use case) - uses official MCP C# SDK with HTTP transport
dotnet run --project MCP/Injector/SnoopWpfMcpServer.csproj

# Run test WPF application
dotnet run --project MCP/TestApp/TestApp.csproj
```

### Running Integration Tests
```bash
# Build and run integration tests
dotnet build MCP/IntegrationTests/IntegrationTests.csproj
dotnet test MCP/IntegrationTests/IntegrationTests.csproj

# Run tests with verbose output
dotnet test MCP/IntegrationTests/IntegrationTests.csproj --logger:console;verbosity=detailed
```

## Architecture Overview

This is a Model Context Protocol (MCP) infrastructure for WPF application inspection and automation. The system uses SnoopWPF injection techniques to communicate with running WPF processes.

### Core Components

1. **MCP/Injector** (SnoopWpfMcpServer) - The main MCP server that exposes WPF inspection capabilities to AI agents
   - Implements MCP protocol via HTTP with both JSON-RPC and REST endpoints
   - Uses Microsoft.SemanticKernel with modern `Kernel.CreateBuilder()` pattern for function definitions
   - Provides process discovery, injection, and communication orchestration

2. **MCP/WpfInspector** - Injectable .NET library that runs inside target WPF processes
   - Creates Named Pipe server for inter-process communication
   - Provides visual tree inspection, UI automation, and screenshot capabilities
   - Implements the `Inspector` class with core inspection logic

3. **MCP/TestApp** - Simple WPF test application for development and testing

4. **MCP/IntegrationTests** - NUnit integration tests that verify end-to-end functionality
   - Starts TestApp and Injector processes
   - Tests HTTP API endpoints for process discovery and visual tree inspection
   - Validates responses against expected JSON structure

5. **snoopwpf/** - Git submodule containing SnoopWPF infrastructure
   - Provides injection capabilities via `Snoop.InjectorLauncher`
   - Contains native C++ injector (`Snoop.GenericInjector`)
   - Core WPF inspection libraries (`Snoop.Core`)

### Communication Flow
```
MCP Client <-> HTTP <-> MCP Server (SnoopWpfMcpServer) <-> Named Pipes <-> WpfInspector (injected into WPF app)
```

### MCP Endpoints
- `GET /mcp/initialize` - Initialize MCP session
- `GET /mcp/tools` - List available tools
- `POST /mcp/tools/{toolName}` - Call a specific tool (REST-style)
- `POST /mcp/rpc` - JSON-RPC endpoint
- `GET /mcp/health` - Health check
- `GET /swagger` - API documentation (development only)

### Key MCP Functions
- `get_wpf_processes`: Discover running WPF applications
- `ping`: Establish communication with target process
- `invoke_automation_peer`: Execute automation peer actions on UI elements using Windows UI Automation patterns
- `get_visual_tree`: Retrieve complete UI hierarchy as JSON with optimized DataContext handling
- `get_element_by_hashcode`: Retrieve a single UI element by type and hashcode (much faster than get_visual_tree for targeted inspection)
- `take_wpf_screenshot`: Capture application screenshots

#### get_element_by_hashcode Usage
The `get_element_by_hashcode` function allows targeted retrieval of a single UI element without the overhead of scanning the entire visual tree. This is particularly useful for checking element state changes after performing automation actions:

```json
{
  "processId": 1234,
  "type": "System.Windows.Controls.Button",
  "hashcode": 567890
}
```

**Use Cases:**
- Check element state after clicking a button
- Verify property changes after automation actions
- Monitor specific control values during testing
- Efficiently inspect individual elements without full tree overhead

**Performance:** Much faster than `get_visual_tree` when you only need one specific element.

### Visual Tree JSON Structure
The `get_visual_tree` function returns a JSON structure with optimized DataContext handling:

```json
{
  "success": true,
  "visualTree": {
    "type": "MainWindow",
    "hashCode": 12345,
    "dataContextId": "dc_67890",  // Only when different from parent
    "properties": {
      "Width": {
        "type": "value",
        "value": 800
      },
      "Height": {
        "type": "binding",
        "path": "WindowHeight",
        "source": "DataContext",
        "mode": "TwoWay",
        "resolvedValue": 600,
        "hasError": false
      },
      "Title": {
        "type": "binding", 
        "path": "WindowTitle",
        "source": "DataContext",
        "mode": "OneWay",
        "hasError": true,
        "error": "Cannot resolve property 'WindowTitle' on DataContext"
      }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [plop44/SnoopWpfMcp](https://github.com/plop44/SnoopWpfMcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
