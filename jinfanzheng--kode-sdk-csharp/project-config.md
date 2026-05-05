---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

The Kode Agent SDK for C# is a sophisticated, event-driven AI Agent runtime built on .NET 10. It provides a comprehensive framework for building AI applications with tool execution, state persistence, multi-agent collaboration, and MCP (Model Context Protocol) integration.

### Key Features
- **Multi-model Support**: Anthropic Claude and OpenAI GPT models
- **Event-Driven Architecture**: Three-channel event system (Progress, Control, Monitor)
- **Tool System**: 20+ built-in tools with custom tool development support
- **State Persistence**: JSON file storage and Redis distributed storage
- **Permission Control**: Fine-grained tool permission management
- **Skills System**: Progressive skill discovery and activation
- **Sub-Agent Delegation**: Task delegation to specialized sub-agents
- **MCP Integration**: Native Model Context Protocol support
- **Source Generator**: Compile-time tool schema generation

## Build, Test, and Development Commands

### Prerequisites
- .NET 10.0 (specified in global.json)
- API keys for Anthropic or OpenAI (for examples that use models)

### Build Commands
```bash
# Build all projects
dotnet build

# Build with release configuration
dotnet build --configuration Release

# Build specific project
dotnet build src/Kode.Agent.Sdk/Kode.Agent.Sdk.csproj

# Clean build artifacts
dotnet clean
```

### Test Commands
```bash
# Run all tests
dotnet test

# Run tests with specific filter
dotnet test --filter "TestCategory=Unit"

# Run tests with coverage
dotnet test --collect:"XPlat Code Coverage"

# Run tests with verbose output
dotnet test --logger "console;verbosity=detailed"
```

### Example Project Commands
```bash
# Run console examples (requires API keys)
cd examples/Kode.Agent.Examples
cp .env.example .env  # Edit with your API keys
dotnet run

# Run WebAPI example (OpenAI SSE compatible)
cd examples/Kode.Agent.WebApiAssistant
cp .env.example .env  # Edit with your API keys
dotnet run
```

### Package Management
```bash
# Restore all packages
dotnet restore

# Central package management (using Directory.Packages.props)
dotnet restore --force-evaluate
```

## High-Level Architecture

### Core Components

#### 1. Agent Core (`Kode.Agent.Sdk/Core/`)
- **Agent.cs**: Main agent implementation with event-driven architecture
- **EventBus.cs**: Three-channel event system (Progress, Control, Monitor)
- **MessageQueue.cs**: Message processing and queueing
- **BreakpointManager.cs**: State persistence and recovery
- **PermissionManager.cs**: Tool permission and approval control

#### 2. Infrastructure (`Kode.Agent.Sdk/Infrastructure/`)
- **Providers/**: Model providers (AnthropicProvider, OpenAIProvider)
- **Sandbox/**: Command execution environment (LocalSandbox)

#### 3. Tools System (`Kode.Agent.Sdk/Tools/`)
- **ToolRegistry.cs**: Tool registration and discovery
- **ToolBase.cs**: Base class for tool development
- **ToolAttributes.cs**: Tool metadata and attributes

#### 4. Storage (`Kode.Agent.Store.*`)
- **JsonAgentStore**: File-based storage (./.kode/agent-id/)
- **RedisAgentStore**: Redis distributed storage

#### 5. Source Generator (`Kode.Agent.SourceGenerator/`)
- Compiles tool schemas at build time (no reflection overhead)

#### 6. MCP Integration (`Kode.Agent.Mcp/`)
- **McpClientManager.cs**: MCP protocol client
- **McpToolProvider.cs**: Tool provider for MCP servers

### Key Patterns and Conventions

#### 1. Event-Driven Architecture
```csharp
// Three-channel event system
await foreach (var envelope in agent.EventBus.SubscribeAsync(EventChannel.Progress))
{
    switch (envelope.Event)
    {
        case TextChunkEvent textChunk:
            Console.Write(textChunk.Delta);  // Streaming output
            break;
        case ToolStartEvent toolStart:
            Console.WriteLine($"[tool] {toolStart.Call.Name} starting...");
            break;
        case DoneEvent done:
            Console.WriteLine("Conversation complete");
            break;
    }
}
```

#### 2. Tool Development with Source Generator
```csharp
[Tool("database_query")]
[Description("Execute SQL query")]
public partial class DatabaseQueryTool : ITool
{
    [ToolParameter("query", required: true)]
    public string Query { get; set; } = "";

    public async Task<ToolResult> ExecuteAsync(ToolContext context)
    {
        // Tool implementation
        return ToolResult.Success(result);
    }
}
```

#### 3. Dependency Injection
```csharp
// Services registration
services.AddKodeAgent(options =>
{
    options.DefaultModel = "claude-sonnet-4-20250514";
    options.StoreDirectory = "./.kode";
});

services.AddAnthropicProvider(options =>
{
    options.ApiKey = Configuration["Anthropic:ApiKey"]!;
});
```

#### 4. State Persistence
```csharp
// Agent automatically persists state during execution
await agent.RunAsync("执行任务");

// Create snapshot for safe branching point
var snapshotId = await agent.SnapshotAsync("backup");

// Resume from store
var restoredAgent = await Agent.ResumeFromStoreAsync("agent-id", deps);
```

## Important Configuration Files

### 1. `global.json`
- Defines .NET 10.0 SDK requirement

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JinFanZheng/kode-sdk-csharp](https://github.com/JinFanZheng/kode-sdk-csharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
