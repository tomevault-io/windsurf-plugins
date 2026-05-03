---
trigger: always_on
description: An IntelliJ Platform plugin that exposes an MCP (Model Context Protocol) server, enabling coding agents to leverage the IDE's powerful indexing and refactoring capabilities.
---

# IDE Index MCP Server - Development Guide

An IntelliJ Platform plugin that exposes an MCP (Model Context Protocol) server, enabling coding agents to leverage the IDE's powerful indexing and refactoring capabilities.

**Works with JetBrains IDEs**: IntelliJ IDEA, PyCharm, WebStorm, GoLand, PhpStorm, RubyMine, CLion, RustRover, DataGrip, and Android Studio.

## Project Overview

### Goal
Create an MCP server within an IntelliJ plugin that allows AI coding assistants to:
- Perform refactoring operations (rename, extract, move, etc.)
- Query type hierarchy and call hierarchy
- Access code navigation features (find usages, find definition)
- Leverage IDE indexes for fast code search and analysis
- Use code completion and inspection APIs

### Technology Stack
- **Language**: Kotlin (JVM 21)
- **Build System**: Gradle 9.0 with Kotlin DSL
- **IDE Platform**: IntelliJ IDEA 2025.1+ (platformType = IC)
- **HTTP Server**: Ktor CIO 2.3.12 (embedded, configurable port)
- **Protocol**: Model Context Protocol (MCP) 2025-03-26

## Key Documentation

### IntelliJ Platform SDK
- **Main Documentation**: https://plugins.jetbrains.com/docs/intellij/welcome.html
- **PSI (Program Structure Interface)**: https://plugins.jetbrains.com/docs/intellij/psi.html
- **Indexing and PSI Stubs**: https://plugins.jetbrains.com/docs/intellij/indexing-and-psi-stubs.html
- **Rename Refactoring**: https://plugins.jetbrains.com/docs/intellij/rename-refactoring.html
- **Modifying the PSI**: https://plugins.jetbrains.com/docs/intellij/modifying-psi.html
- **Plugin Configuration**: https://plugins.jetbrains.com/docs/intellij/plugin-configuration-file.html
- **Explore API**: https://plugins.jetbrains.com/docs/intellij/explore-api.html

### Model Context Protocol (MCP)
- **Specification**: https://spec.modelcontextprotocol.io/specification/2025-03-26/
- **Tools API**: https://modelcontextprotocol.io/specification/2025-03-26/server/tools
- **Resources API**: https://modelcontextprotocol.io/specification/2025-03-26/server/resources
- **Legacy SSE Transport**: https://spec.modelcontextprotocol.io/specification/2024-11-05/basic/transports/
- **GitHub**: https://github.com/modelcontextprotocol/modelcontextprotocol

## Project Structure

```
src/
├── main/
│   ├── kotlin/com/github/hechtcarmel/jetbrainsindexmcpplugin/
│   │   ├── MyBundle.kt                 # Resource bundle accessor
│   │   ├── handlers/                   # Language-specific handlers
│   │   │   ├── LanguageHandler.kt      # Handler interfaces & data classes
│   │   │   ├── LanguageHandlerRegistry.kt # Data-driven handler registry
│   │   │   ├── OptimizedSymbolSearch.kt # Symbol search using platform APIs
│   │   │   ├── java/JavaHandlers.kt    # Java/Kotlin handlers
│   │   │   ├── python/PythonHandlers.kt # Python handlers (reflection)
│   │   │   ├── javascript/JavaScriptHandlers.kt # JS/TS handlers (reflection)
│   │   │   ├── go/GoHandlers.kt        # Go handlers (reflection)
│   │   │   ├── php/PhpHandlers.kt      # PHP handlers (reflection)
│   │   │   └── rust/RustHandlers.kt    # Rust handlers (reflection)
│   │   ├── server/                     # MCP server infrastructure
│   │   │   ├── McpServerService.kt     # App-level service managing server lifecycle
│   │   │   ├── JsonRpcHandler.kt       # JSON-RPC 2.0 request routing
│   │   │   ├── ProjectResolver.kt      # Multi-project resolution with workspace support
│   │   │   ├── models/                 # Protocol models (JsonRpc, MCP)
│   │   │   └── transport/              # HTTP+SSE transport layer
│   │   │       ├── KtorMcpServer.kt    # Embedded Ktor CIO server
│   │   │       ├── KtorSseSessionManager.kt # SSE session management
│   │   ├── startup/                    # Startup activities
│   │   ├── tools/                      # MCP tool implementations
│   │   │   ├── McpTool.kt             # Tool interface
│   │   │   ├── AbstractMcpTool.kt     # Base class (PSI sync, threading, helpers)
│   │   │   ├── ToolRegistry.kt        # Data-driven tool registry
│   │   │   ├── schema/                # Tool schema utilities
│   │   │   │   └── SchemaBuilder.kt   # Fluent builder for input schemas
│   │   │   ├── editor/                # Editor interaction tools
│   │   │   ├── navigation/            # Navigation tools (multi-language)
│   │   │   ├── intelligence/          # Code analysis tools
│   │   │   ├── project/               # Project status tools
│   │   │   └── refactoring/           # Refactoring tools
│   │   ├── util/                      # Utilities
│   │   │   ├── PluginDetector.kt      # Generic plugin availability detector
│   │   │   ├── PluginDetectors.kt     # Registry of all language detectors
│   │   │   ├── ClassResolver.kt       # Class lookup by FQN (Java, PHP)
│   │   │   ├── ProjectUtils.kt        # Project/workspace helpers
│   │   │   ├── PsiUtils.kt            # PSI navigation helpers
│   │   │   └── ThreadingUtils.kt      # Threading utilities
│   │   └── ui/                        # Tool window UI
│   └── resources/
│       ├── META-INF/
│       │   ├── plugin.xml              # Plugin configuration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hechtcarmel/jetbrains-index-mcp-plugin](https://github.com/hechtcarmel/jetbrains-index-mcp-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
