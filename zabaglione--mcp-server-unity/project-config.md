---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Unity MCP Server - Project Knowledge Base

## Project Overview
Unity MCP Server is a Model Context Protocol (MCP) server that bridges AI assistants (like Claude) with Unity game development. It supports both legacy file-based operations (v2.x) and direct Unity Editor integration (v3.0.0 for Unity 6000+).

## Commands

### Build and Development
- `npm run build` - Compile TypeScript to JavaScript
- `npm run dev` - Watch mode for development
- `npm start` - Start MCP server (stdio mode for Claude Desktop)
- `npm run start:http` - Start HTTP API server (default port 3000)
- `npm run clean` - Clean build artifacts

### Optimized Mode
- `npm run start:optimized` - MCP server with streaming for large files
- `npm run start:http:optimized` - HTTP server with streaming

### Testing
- `npm test` - Run all tests
- `npm run test:unit` - Unit tests only
- `npm run test:integration` - Integration tests only  
- `npm run test:e2e` - End-to-end tests
- `npm run test:coverage` - Generate coverage report
- `npm run test:manual` - Interactive manual test runner
- `npm run test:performance` - Run performance benchmarks
- `npm run test:watch` - Watch mode for tests
- `npm run test:legacy` - Old integration test
- `npm run test:direct` - Direct TypeScript test runner

### No Linting/Formatting
- No ESLint or Prettier configured - maintain existing code style

## Architecture Evolution

### v3.0.0 - Unity Bridge Architecture (Unity 6000+ only)
- **Direct Unity API Integration**: No file system manipulation
- **Unity Bridge Client**: Named Pipes/Domain Sockets communication
- **Simplified API**: Focus on script and folder operations only
- **Real-time Events**: Connection, compilation, project changes
- **Entry Points**: `src/index.ts`, `src/unity6-mcp-server.ts`
- **Unity Script**: `src/unity-scripts/MCPBridge.cs` (place in Unity project)

### v2.x - Service-Based Architecture (Legacy, Unity 2019.4+)
All services extend from `BaseService` and follow a consistent pattern:
- **ProjectService**: Unity project validation and setup
- **ScriptService**: C# script creation and management  
- **AssetService**: Unity asset reading and listing
- **BuildService**: Multi-platform build automation
- **ShaderService**: Shader creation for Built-in/URP/HDRP
- **MaterialService**: Material creation and property management
- **EditorScriptService**: Editor extensions (windows, inspectors)
- **CodeAnalysisService**: Code diff, namespace management, duplicate detection
- **CompilationService**: Real-time compilation error monitoring
- **UnityRefreshService**: Asset database refresh with batch operations
- **UnityDiagnosticsService**: Editor log analysis and error tracking
- **UIToolkitService**: UXML/USS file creation and management

### Key Design Patterns

1. **Service Container Pattern** (v2.x)
   - All services registered in `ServicesContainer`
   - Dependency injection for service dependencies
   - Factory pattern for service instantiation

2. **Template-based Code Generation**
   - Templates in `src/templates/` for all generated code
   - Supports shader variants (Built-in, URP, HDRP)
   - Namespace auto-detection based on file location

3. **Meta File Management** (v2.x critical)
   - Automatic .meta file generation with consistent GUIDs
   - GUID preservation for shader/material updates
   - Prevents Unity reference breakage

4. **Render Pipeline Detection**
   - Auto-detects Built-in, URP, or HDRP from project packages
   - Adjusts shader/material creation accordingly

## Development Guidelines

### Error Handling
- All services use custom error types from `src/errors/`
- Detailed error messages with actionable suggestions
- Validation before operations (project path, Unity version)

### File Operations
- Always use absolute paths
- Create parent directories automatically
- Generate .meta files for all Unity assets (v2.x)
- Respect Unity's folder structure conventions

### Code Style
- TypeScript with strict type checking
- ES modules with .js extensions in imports
- Async/await for all I/O operations
- Comprehensive logging with context

### Testing
- Jest framework with TypeScript support
- Virtual Unity project utility for test environments (`tests/utils/virtualUnityProject.ts`)
- Snapshot testing for generated content validation
- Performance benchmarks exported to JSON
- Coverage thresholds: 80% lines, 70% branches/functions
- Test structure mirrors source structure (e.g., `src/services/foo.ts` → `tests/unit/services/foo.test.ts`)

## Unity Integration Points

### Project Structure Expected
```
UnityProject/
├── Assets/
│   ├── Scripts/
│   ├── Materials/
│   ├── Shaders/
│   └── Editor/
│       └── MCP/
│           └── MCPBridge.cs  # v3.0 Unity Bridge script
├── Packages/
│   └── manifest.json (render pipeline detection)
├── Library/
│   ├── Bee/fullprofile.json (compilation errors)
│   └── Logs/ (Unity console logs)
└── ProjectSettings/
```

### Compilation Monitoring
- Watches `Library/Bee/fullprofile.json` for errors
- Parses Unity console logs from `Library/Logs/`
- Real-time feedback on script compilation

### Asset Database Refresh (v2.x)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zabaglione/mcp-server-unity](https://github.com/zabaglione/mcp-server-unity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
