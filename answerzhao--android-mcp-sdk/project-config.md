---
trigger: always_on
description: Provides two tools with configurable security policies:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an **Android Binder MCP SDK** - a Model Context Protocol implementation using Android's Binder IPC mechanism. The SDK enables Large Language Models (LLMs) to securely call native Android app functions (tools) across process boundaries.

## Key Architecture

The SDK follows a layered architecture:
```
LLM Application → MCPApi → MCPClientManager → MCPClient → BinderClientTransport → AIDL/Binder → BinderServerTransport → MCPServer → MCPService
```

### Core Modules

- **`sdk/`**: Core MCP Binder SDK library (main implementation)
- **`android-mcp-server/`**: Demo weather service providing MCP tools
- **`android-mcp-client/`**: Demo client app with OpenAI LLM integration

## Essential Commands

### Build Commands
```bash
# Build entire project
./gradlew build

# Build specific modules
./gradlew :sdk:build
./gradlew :android-mcp-server:assembleDebug
./gradlew :android-mcp-client:assembleDebug

# Run tests
./gradlew test
./gradlew :sdk:test
```

### Testing Commands
```bash
# Unit tests
./gradlew testDebugUnitTest

# Android instrumented tests  
./gradlew connectedDebugAndroidTest

# Specific module tests
./gradlew :sdk:testDebugUnitTest
```

### Development Commands
```bash
# Clean build
./gradlew clean

# Install debug APKs (requires connected device/emulator)
./gradlew :android-mcp-server:installDebug
./gradlew :android-mcp-client:installDebug

# Check dependencies
./gradlew dependencies
```

## Key Technologies & Dependencies

- **Kotlin 2.2.0** with coroutines
- **MCP Kotlin SDK 0.6.0** (`io.modelcontextprotocol:kotlin-sdk`)
- **Kotlinx Serialization JSON 1.7.3** for JSON handling
- **Android Gradle Plugin 8.10.0**
- **MinSDK 24, CompileSDK 36**

## Core SDK Architecture

### Transport Layer (`sdk/src/main/java/zwdroid/mcp/sdk/transport/`)
- **`BinderServerTransport`**: Server-side IPC with linkToDeath mechanism
- **`BinderClientTransport`**: Client-side IPC with service discovery

### Server Components (`sdk/src/main/java/zwdroid/mcp/sdk/server/`)
- **`MCPService`**: Abstract base class for tool providers (extend this)
- **`MCPServer`**: Handles MCP request processing

### Client Components (`sdk/src/main/java/zwdroid/mcp/sdk/client/`)
- **`MCPApi`**: High-level interface for LLM integration
- **`MCPClientManager`**: Service discovery and lifecycle management
- **`MCPClient`**: Individual service connections

### AIDL Interfaces (`sdk/src/main/aidl/zwdroid/mcp/sdk/`)
- **`IMCPBinderService.aidl`**: Main service interface
- **`IMCPBinderCallback.aidl`**: Async response callbacks

### Common Components (`sdk/src/main/java/zwdroid/mcp/sdk/common/`)
- **`PermissionPolicy`**: Security policy enumeration (OPEN, PERMISSION_BASED, etc.)
- **`SecurityUtils`**: Security utilities for signature validation and caller verification
- **`McpExceptions`**: Comprehensive exception hierarchy for error handling
- **`KLog`**: Android logging wrapper with debug capabilities

## Security Model

The SDK implements a **unified permission system** with multiple security strategies:

### Standard MCP Permission

The SDK defines a single, standard permission that all MCP services use:
- **Permission**: `zwdroid.mcp.sdk.permission.ACCESS_MCP_SERVICES`
- **Protection Level**: `normal`
- **Defined in**: SDK's AndroidManifest.xml

### Permission Policies
Services can choose from 5 permission policies by overriding `getPermissionPolicy()`:

- **`PERMISSION_BASED`** (default): Requires clients to have the standard MCP SDK permission
- **`OPEN`**: Allows any client to access the service - for development/testing only
- **`SIGNATURE_BASED`**: Only allows apps signed with the same certificate
- **`WHITELIST_BASED`**: Only allows specific package names (configured via `getAllowedPackages()`)
- **`CUSTOM`**: Uses service-specific validation logic (via `validateCustomCaller()`)

### Security Implementation
- **Unified permissions**: All services use the same standard permission by default
- **Application-level control**: Permission validation applies to entire service, not per tool
- **Real-time validation**: Access checked on each tool call, not just service binding
- **Caller identification**: Uses `Binder.getCallingUid()` and `Binder.getCallingPid()` in proper Binder call context
- **Context-aware validation**: Caller information captured before entering coroutine scope to preserve Binder context
- **Comprehensive logging**: Detailed permission validation logs for debugging
- **Standard error responses**: Returns JSON-RPC compliant error messages for denied access

### Security Utilities
- **`SecurityUtils.kt`**: Signature validation, package verification, security logging
- **`PermissionPolicy.kt`**: Enumeration of available security strategies
- **Enhanced caller validation**: Package whitelist/blacklist support

### Example Usage
```kotlin
class WeatherMCPService : MCPService() {
    // Default: Use standard MCP permission (PERMISSION_BASED)
    // No override needed - this is the default behavior
    
    // Alternative: Use whitelist-based security
    override fun getPermissionPolicy() = PermissionPolicy.WHITELIST_BASED
    override fun getAllowedPackages() = setOf(

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AnswerZhao/android-mcp-sdk](https://github.com/AnswerZhao/android-mcp-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
