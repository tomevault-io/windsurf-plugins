---
trigger: always_on
description: This is a multi-module Android application that implements a **Server-Sent Events (SSE) MCP (Model Context Protocol) server**. The server enables AI assistants to connect and interact with Android device capabilities through a standardized protocol.
---

# Android MCP Server - Instructions

## Project Overview

This is a multi-module Android application that implements a **Server-Sent Events (SSE) MCP (Model Context Protocol) server**. The server enables AI assistants to connect and interact with Android device capabilities through a standardized protocol.

### Key Features
- **SSE-based MCP Server**: Runs as an Android foreground service with Ktor
- **Modular Tool Architecture**: Each device capability is a separate, pluggable module
- **Dynamic Tool Management**: Tools can be enabled/disabled at runtime with user consent
- **Privacy-Focused**: Each tool requires explicit user permission with clear disclaimers
- **Authentication**: Bearer token-based authentication for secure connections
- **External Tool Support**: Third-party apps can expose MCP tools via Content Providers

## Architecture Overview

The application follows a **clean, modular architecture** with clear separation of concerns:

```
android-mcp-server/
├── app/                    # Main application module
├── core/                   # Core interfaces and contracts
├── mcp-provider/          # Library for external tool integration
├── externalmcptool/       # Example external tool implementation
└── tools/                 # Individual tool modules
    ├── sms/
    ├── camera/
    ├── contacts/
    ├── sensor/
    ├── ads/
    ├── smsintent/
    └── externaltools/
```

### Core Modules

#### 1. `/app` - Main Application Module
**Purpose**: Contains the UI, server logic, and service management

**Key Components**:
- `McpServerService`: Foreground service that runs the Ktor SSE server
- `McpServerApplication`: Application class with Hilt setup
- `MainActivity`: Compose UI for server control and tool management
- `ToolService`: Manages tool states and preferences
- `ToolPreferencesRepository`: DataStore-based persistence for tool states
- `AuthRepository`: Manages authentication tokens

**Dependencies**: All tool modules, core, mcp-provider

#### 2. `/core` - Core Interfaces
**Purpose**: Defines the `McpTool` interface that all tools implement

**Key Interface**:
```kotlin
interface McpTool {
    val id: String                          // Unique identifier
    val name: String                        // Display name
    val enabledByDefault: Boolean           // Default enabled state
    val disclaim: String?                   // Privacy/warning disclaimer
    fun configure(server: Server)           // Register with MCP server
    fun requiredPermissions(): Set<String>  // Android permissions needed
}
```

**Type**: Java library module (not Android library)

#### 3. `/mcp-provider` - External Tool SDK
**Purpose**: Library that external Android apps can use to expose MCP tools

**Key Classes**:
- `McpProvider`: Base class for creating external tool providers
- `ToolInfo`, `Tools`: Data classes for tool metadata
- `ToolInput`: Various input type definitions

**Integration Method**: Content Provider with custom authority

#### 4. `/tools/*` - Tool Modules
**Purpose**: Individual modules implementing specific device capabilities

**Current Tools**:
- `sms`: Send SMS messages
- `smsintent`: Send SMS via Intent (no permission required)
- `camera`: Access camera and take photos
- `contacts`: Read contacts
- `sensor`: Access device sensors
- `ads`: Display ads (example tool)
- `externaltools`: Discovers and integrates external MCP tools

## Tool Module Structure

Each tool module follows a **consistent, standardized structure** for maintainability:

### Standard Directory Layout
```
tools/{LOWERCASE_TOOL_NAME}/
├── build.gradle.kts
├── src/main/
    ├── AndroidManifest.xml
    └── java/se/premex/mcp/{LOWERCASE_TOOL_NAME}/
        ├── configurator/
        │   ├── {Toolname}ToolConfigurator.kt      # Interface
        │   └── {Toolname}ToolConfiguratorImpl.kt  # Implementation
        ├── di/
        │   └── {Toolname}Tool.kt                  # Hilt module
        ├── repositories/
        │   ├── {Toolname}Info.kt                  # Data class
        │   ├── {Toolname}Repository.kt            # Interface
        │   └── {Toolname}RepositoryImpl.kt        # Implementation
        └── tool/
            └── {Toolname}Tool.kt                  # McpTool implementation
```

### Two Main Architectural Patterns

#### Pattern 1: Configurator-Based (Recommended for complex tools)
**Example**: camera, contacts

**Structure**:
- **Configurator Interface**: Defines `configure(server: Server)`
- **Configurator Implementation**: Registers MCP tools with the server
- **Repository**: Handles Android API interactions
- **Tool Class**: Implements `McpTool`, delegates to configurator

**When to use**: Tools with multiple MCP endpoints, complex logic, or significant Android API interaction

#### Pattern 2: Direct Configuration (Simpler tools)
**Example**: sms

**Structure**:
- **Extension Function**: `appendXxxTools(server: Server, dependency)`
- **Service/Repository**: Business logic
- **Tool Class**: Implements `McpTool`, calls extension function

**When to use**: Simple tools with single or few MCP endpoints


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [premex-ab/phone-mcp](https://github.com/premex-ab/phone-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
