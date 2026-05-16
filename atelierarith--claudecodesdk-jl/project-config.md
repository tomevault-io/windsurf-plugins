---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Julia Development Commands

### Testing

If you modify any source code, please run the following testing protocols:

```bash
# Run all tests (288 tests total)
julia --project -e "using Pkg; Pkg.test()"

# Run specific test files
julia --project test/test_types.jl
julia --project test/test_errors.jl
julia --project test/test_client.jl
julia --project test/test_transport.jl
julia --project test/test_integration.jl
```

### Documentation

After making code changes, update the documentation:

```
# Update documentation files in docs/src/ if needed
# Key files:
# - docs/src/index.md - Main landing page
# - docs/src/getting-started.md - Installation and basic usage
# - docs/src/examples.md - Comprehensive usage examples
# - docs/src/api.md - API reference (auto-generated from docstrings)

# Note: All documentation examples use keyword argument syntax:
# query(prompt="...", options=options) - CORRECT
# query("...") - INCORRECT (will cause MethodError)
```

Then, run the following commands

```bash
julia --project=docs -e 'using Pkg; Pkg.develop(path=".")'
julia --project=docs docs/make.jl
```

### Example Usage
```bash
# Run example files
julia --project examples/quick_start.jl
julia --project examples/streaming_demo.jl
julia --project examples/tool_execution_demo.jl
julia --project examples/cli_aware_demo.jl
```

## Architecture Overview

This is an **unofficial** Julia SDK for Claude Code that **fully mirrors** the Python SDK architecture and functionality. The implementation is complete and provides the same capabilities as the official Python SDK while leveraging Julia's type safety and performance.

### Core Components

1. **Main Entry Point** (`src/ClaudeCodeSDK.jl`):
   - Exports `query(; prompt, options)` function with keyword arguments matching Python SDK
   - Manages `InternalClient` and sets environment variables

2. **Type System** (`src/types.jl`):
   - Complete type definitions matching Python SDK structure
   - `ClaudeCodeOptions` with all 14 configuration fields
   - Message types: `AssistantMessage`, `UserMessage`, `SystemMessage`, `ResultMessage`
   - Content blocks: `TextBlock`, `ToolUseBlock`, `ToolResultBlock`
   - MCP support: `McpServerConfig`

3. **Client Layer** (`src/internal/client.jl`):
   - `InternalClient` orchestrates queries and parses responses
   - Message parsing from CLI JSON output into typed structures

4. **Transport Layer** (`src/internal/cli.jl`):
   - `SubprocessCLITransport` handles CLI communication
   - Comprehensive CLI discovery and command building
   - JSON streaming support with line-by-line parsing
   - Robust process management

5. **Error Handling** (`src/errors.jl`):
   - Complete exception hierarchy matching Python SDK
   - `ClaudeSDKError`, `CLINotFoundError`, `CLIConnectionError`, `ProcessError`, `CLIJSONDecodeError`

6. **Tool System** (`src/internal/tools.jl`): Local tool execution functionality

### Key Features Implemented

✅ **Complete Python SDK Compatibility:**
- **API**: Keyword argument API `query(prompt="...", options=...)`
- **Configuration**: All 14 `ClaudeCodeOptions` fields supported
- **CLI Integration**: Full command building with all CLI options
- **MCP Support**: Model Context Protocol servers and tools
- **Error Handling**: Complete exception hierarchy
- **Message Types**: All message and content block types
- **Tool Execution**: Read, Write, Bash tools with proper results
- **Cost Tracking**: Usage and cost information
- **Environment**: Working directory and environment variable support

✅ **Julia-Specific Enhancements:**
- **Type Safety**: Strong typing throughout with union types
- **Performance**: Efficient subprocess handling
- **Error Messages**: Clear, actionable error descriptions
- **Documentation**: Comprehensive inline documentation

## Prerequisites

- Julia 1.10+
- Claude Code CLI installed: `npm install -g @anthropic-ai/claude-code`
- The SDK automatically detects CLI availability and provides helpful error messages

## Testing Strategy

**Comprehensive test suite with 288 tests total** - All ported from Python SDK:

### Test Files Structure:
1. **`test/test_types.jl`** - Message types, options configuration, content blocks
2. **`test/test_errors.jl`** - Error hierarchy, exception handling, string representations
3. **`test/test_client.jl`** - Query function, message processing, client configuration
4. **`test/test_transport.jl`** - CLI discovery, command building, JSON streaming, process management
5. **`test/test_integration.jl`** - End-to-end testing, CLI integration, comprehensive scenarios

### Test Coverage:
- **Type Construction**: Test SDK component creation and validation
- **Message Types**: Test all message and content block types
- **Tool Functionality**: Test tool creation and execution
- **CLI Integration**: Full end-to-end testing with actual CLI (when available)
- **Error Handling**: Test all exception types and scenarios
- **JSON Streaming**: Test CLI response parsing and message flow
- **Process Management**: Test subprocess lifecycle and error handling

### Test Environment Handling:
Tests gracefully handle CLI availability:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AtelierArith/ClaudeCodeSDK.jl](https://github.com/AtelierArith/ClaudeCodeSDK.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
