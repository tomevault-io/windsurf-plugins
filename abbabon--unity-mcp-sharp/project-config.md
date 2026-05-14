---
trigger: always_on
description: This document provides development guidelines, workflow instructions, and architectural details for contributors and AI assistants working on the Unity MCP Server project.
---

# Unity MCP Server - Development Guide

This document provides development guidelines, workflow instructions, and architectural details for contributors and AI assistants working on the Unity MCP Server project.

## Project Overview

Unity MCP Server is a **dual-purpose project**:

1. **Unity Package** (UPM) - Installed in Unity projects via OpenUPM
2. **MCP Server** (Dockerized .NET application) - Runs in a container, communicates with Unity

### Key Technologies

- **.NET 9.0** - Server runtime
- **ASP.NET Core** - Web framework for SSE and WebSocket
- **Model Context Protocol SDK** - Official C# MCP implementation
- **Unity 2021.3+** - Target Unity version
- **UIToolkit** - Unity's modern UI framework
- **Docker** - Containerization
- **GitHub Actions** - CI/CD pipeline

## Repository Structure

```
unity-mcp-sharp/
├── .github/
│   └── workflows/           # CI/CD pipelines
│       ├── build-server.yml
│       ├── publish-docker.yml
│       └── publish-openupm.yml
│
├── Server~/                 # MCP Server (excluded from Unity package via ~)
│   ├── Models/              # JSON-RPC message models
│   ├── Services/            # WebSocket service, handlers
│   ├── Tools/               # MCP tool implementations
│   ├── Program.cs           # Server entry point
│   ├── Dockerfile
│   ├── docker-compose.yml
│   └── UnityMcpServer.csproj
│
├── Scripts~/                # Development scripts (excluded from Unity package via ~)
│   ├── rebuild.sh           # Build .NET server + Docker image
│   ├── sign-package.sh      # Sign UPM package for Unity 6+
│   ├── start-mcp-server.sh  # Start MCP server container
│   └── test.sh              # Smoke test script
│
├── Runtime/                 # Unity runtime scripts
│   ├── Scripts/
│   │   ├── MCPClient.cs     # WebSocket JSON-RPC client
│   │   ├── MCPServerManager.cs  # Docker lifecycle management
│   │   ├── MCPConfiguration.cs  # ScriptableObject config
│   │   └── WindowFocusHelper.cs # Platform-specific window focus (P/Invoke)
│   └── UnityMCPSharp.asmdef
│
├── Editor/                  # Unity Editor scripts
│   ├── Scripts/
│   │   ├── MCPDashboard.cs      # UIToolkit EditorWindow
│   │   ├── MCPEditorIntegration.cs  # Handles MCP requests
│   │   ├── MCPMenuItems.cs
│   │   └── DockerSetupWizard.cs
│   ├── UI/                  # UIToolkit assets (UXML, USS)
│   └── UnityMCPSharp.Editor.asmdef
│
├── Documentation~/          # User documentation (excluded from package)
│   ├── Installation.md
│   ├── Configuration.md
│   ├── PackageSigning.md    # Unity 6+ package signing guide
│   ├── Troubleshooting.md
│   ├── Testing.md
│   └── MultiEditor-TechnicalRundown.md  # Technical deep-dive docs
│
├── TestProject~/            # Test Unity project (excluded from package)
│
├── package.json             # Unity UPM manifest
├── README.md                # Main project readme
├── CHANGELOG.md             # Version history
├── CLAUDE.md                # This file - AI assistant instructions
└── LICENSE
```

**Note on folder naming convention:**
- Folders ending with `~/` are excluded from the Unity package
- `Server~/` - .NET MCP server code
- `Scripts~/` - Development/build scripts
- `Documentation~/` - Extended documentation
- `TestProject~/` - Local testing Unity project

## Architecture Details

### Communication Flow

```
LLM → MCP Client (IDE) → [HTTP/SSE] → MCP Server → [WebSocket/JSON-RPC] → Unity Package → Unity Editor APIs
```

### Dual Transport Design

The server supports **two transports simultaneously**:

1. **HTTP (Streamable HTTP)** at `/mcp`
   - For MCP protocol communication with IDEs (Claude Code, Cursor, etc.)
   - Uses the official MCP SDK (ModelContextProtocol.AspNetCore)
   - Supports both modern Streamable HTTP and legacy SSE for backward compatibility

2. **WebSocket** at `/ws`
   - For bidirectional communication with Unity Editor
   - JSON-RPC 2.0 protocol for consistency
   - Persistent connection for real-time updates

### Multi-Editor Support (v0.5.0+)

The server supports **multiple Unity Editor instances** connecting simultaneously:

**Architecture:**
- Each Unity Editor registers with unique metadata (project name, scene, machine, process ID)
- Each MCP session (HTTP connection) can select a different Unity Editor
- Session-to-editor mapping persists across Unity compilation reconnects
- Smart auto-selection: single editor auto-selected, multiple editors require explicit selection

**Key Components:**
- `EditorSessionManager` - Manages session-to-editor mappings using `ConcurrentDictionary`
- `McpSessionContext` - AsyncLocal storage for MCP session IDs (propagates through async chains)
- `McpSessionMiddleware` - ASP.NET middleware captures HTTP connection ID as session ID
- `EditorMetadata` - Rich metadata model for Unity Editor instances

**Session Isolation:**
```
MCP Session A → Unity Editor 1 (ProjectX, SceneA)
MCP Session B → Unity Editor 2 (ProjectY, SceneB)
MCP Session C → Unity Editor 1 (same as Session A)
```

**New MCP Tools:**
- `unity_list_editors` - List all connected editors with metadata
- `unity_select_editor` - Select which editor to use for current session

### Auto-Focus Feature (v0.6.0+)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Abbabon/unity-mcp-sharp](https://github.com/Abbabon/unity-mcp-sharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
