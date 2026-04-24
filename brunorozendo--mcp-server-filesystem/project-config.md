---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Multi-module Java MCP (Model Context Protocol) server providing filesystem access through three different transport mechanisms: stdio, HTTP servlet, and SSE (Server-Sent Events). All implementations share common business logic in the `tools` module and expose 10 filesystem operation tools.

## Project Structure

This is a **multi-module Gradle project** with four modules:

- **`tools/`** - Shared business logic and tool definitions (library)
- **`stdio/`** - Standalone application using stdio transport (JAR)
- **`http/`** - HTTP servlet implementation (WAR)
- **`sse/`** - Server-Sent Events servlet implementation (WAR)

## Build and Development Commands

### Building All Modules
```bash
# Clean and build all modules
./gradlew clean build

# Build without tests
./gradlew build -x test
```

### Building Individual Modules
```bash
# Build specific module
./gradlew :stdio:build
./gradlew :http:build
./gradlew :sse:build
./gradlew :tools:build

# Build native executable (stdio module with GraalVM)
./gradlew :stdio:nativeCompile
```

### Testing
```bash
# Run all tests (using Spock framework in tools module)
./gradlew test

# Run tests for specific module
./gradlew :tools:test

# Generate coverage report (Jacoco)
./gradlew :tools:test jacocoTestReport
```

### Build Artifacts
- stdio: `stdio/build/libs/stdio-1.0.0.jar` (application JAR)
- http: `http/build/libs/http-1.0.0.war` (WAR file)
- sse: `sse/build/libs/sse-1.0.0.war` (WAR file)
- tools: `tools/build/libs/tools-1.0.0.jar` (library JAR)

## Architecture

### Module Architecture

The project follows a **shared library pattern** where the `tools` module contains all business logic and the three transport modules (`stdio`, `http`, `sse`) are thin adapters that wire up the appropriate transport layer.

**Shared Components (tools module)**:

- **`Transport.java`** - Central MCP server configuration that all transport implementations use
  - Provides three static `getMcp()` methods for different transport providers:
    - `getMcp(McpServerTransportProvider)` - Generic provider
    - `getMcp(HttpServletStreamableServerTransportProvider)` - HTTP transport
    - `getMcp(HttpServletSseServerTransportProvider)` - SSE transport
  - Single `buildMcpServer()` method registers all 10 tools with their handlers
  - Server info: name="file-reader-server", version="1.0.0"
  - Server capabilities: is correct, `tools=false` indicate that the list of available tools is fixed and will not grow or decrease, this MCP server does not provide `resources` therefore `resources(false, false)`  indicate the there is no option to subscribe to resources and the list of resources is fixed and will nor grow or decrease.
  - Located at: `tools/src/main/java/com/brunorozendo/mcp/filesystem/Transport.java`

- **`FileTools.java`** - All filesystem operation business logic
  - 10 tool handler methods: `readFile()`, `readMultipleFiles()`, `writeFile()`, `editFile()`, `createDirectory()`, `listDirectory()`, `directoryTree()`, `moveFile()`, `searchFiles()`, `getFileInfo()`
  - Each returns `Mono<CallToolResult>` (reactive pattern using Project Reactor)
  - Uses `handleTool()` helper for consistent error handling
  - All operations use Java NIO `Path` and `Files` APIs
  - **No path validation** - filesystem operations have no directory restrictions

- **`ToolSchemas.java`** - MCP tool schema definitions
  - Defines 10 tool schemas using `McpSchema.Tool` and `McpSchema.JsonSchema`
  - Each tool has: name, display name, description, input schema
  - Schema helper methods: `createSinglePathSchema()`, `createMultiPathSchema()`, `createEditFileSchema()`, etc.
  - Note: Tool descriptions mention "allowed directories" but no path validation exists in the implementation

**Transport Implementations**:

1. **stdio module** (`stdio/src/main/java/com/brunorozendo/mcp/filesystem/FilesystemServer.java`)
   - Standalone Java application with `static void main()` method
   - **BUG**: Method signature is `static void main()` instead of `public static void main(String[] args)` - won't execute
   - Uses `StdioServerTransportProvider` with `JacksonMcpJsonMapper` for stdin/stdout communication
   - Calls `Transport.getMcp(transportProvider).build()` to initialize server
   - Build configuration creates fat JAR with manifest Main-Class attribute
   - Usage (after fixing main): `java -jar stdio-1.0.0.jar`

2. **http module** (`http/src/main/java/com/brunorozendo/mcp/filesystem/FilesystemServer.java`)
   - Servlet extending `HttpServlet` with `@WebServlet` annotation
   - Mapped to `/mcp`, `/mcp/*` with `asyncSupported = true`
   - Uses `HttpServletStreamableServerTransportProvider` built with:
     - `.mcpEndpoint("/mcp")`
     - `.jsonMapper(new JacksonMcpJsonMapper(new ObjectMapper()))`
   - Initializes in `init()` method, delegates to `transportProvider.service(req, resp)` in `service()`
   - Has unused `doGet()` override that calls `super.doGet()`
   - Requires servlet container (Tomcat, Jetty, etc.)

3. **sse module** (`sse/src/main/java/com/brunorozendo/mcp/filesystem/FilesystemServer.java`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brunorozendo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
