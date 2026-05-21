---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

VectorMCP is a Ruby gem implementing the Model Context Protocol (MCP) server-side specification. It provides a framework for creating MCP servers that expose tools, resources, prompts, and roots to LLM clients with comprehensive security features, structured logging, and production-ready capabilities.

## Essential Commands

### Development Setup

```bash
bin/setup          # Install dependencies and setup development environment
bin/console        # Interactive Ruby console with the gem loaded
```

### Testing and Quality

```bash
rake               # Run default task (tests + linting)
rake spec          # Run RSpec test suite only
rake rubocop       # Run RuboCop linting only
bundle exec rspec          # Run RSpec test suite  
bundle exec rspec ./spec/vector_mcp/examples_spec.rb # run a single test file
bundle exec rspec ./spec/vector_mcp/logging_spec.rb # run logging tests
bundle exec rubocop       # Run RuboCop linting and style checks
```

### Documentation

```bash
rake yard          # Generate YARD documentation
rake doc           # Alias for yard (outputs to doc/ directory)
```

### Example Usage

```bash
# Getting Started Examples
ruby examples/getting_started/basic_http_stream_server.rb # MCP-compliant HTTP streaming

# Core Features Examples
ruby examples/core_features/authentication.rb        # Authentication demo
ruby examples/core_features/filesystem_roots.rb      # Filesystem boundaries
ruby examples/core_features/input_validation.rb      # Schema validation
ruby examples/core_features/cli_client.rb           # Command-line client

# Logging Examples
ruby examples/logging/basic_logging.rb              # Component-based logging
ruby examples/logging/structured_logging.rb         # JSON structured output
ruby examples/logging/security_logging.rb           # Security audit logging
ruby examples/logging/log_analysis.rb               # Log processing tools

# Use Cases Examples
ruby examples/use_cases/file_operations.rb          # File system operations
ruby examples/use_cases/data_analysis.rb            # Data processing workflows
ruby examples/use_cases/web_scraping.rb             # Content extraction

# Middleware Examples
ruby examples/middleware_examples.rb                # Comprehensive middleware demo
ruby examples/simple_middleware_demo.rb             # Basic middleware patterns
```

### Logging Configuration

```bash
# Environment-based logging configuration
VECTORMCP_LOG_LEVEL=DEBUG ruby examples/logging_demo.rb
VECTORMCP_LOG_FORMAT=json ruby examples/auth_server.rb
VECTORMCP_LOG_OUTPUT=file VECTORMCP_LOG_FILE_PATH=/tmp/vectormcp.log ruby examples/getting_started/basic_http_stream_server.rb
```

## Architecture

### Core Components

- **VectorMCP::Server** (`lib/vector_mcp/server.rb`): Main server class with modular architecture
  - `Server::Registry` (`lib/vector_mcp/server/registry.rb`): Tool/resource/prompt/root registration
  - `Server::Capabilities` (`lib/vector_mcp/server/capabilities.rb`): Server info and capability management
  - `Server::MessageHandling` (`lib/vector_mcp/server/message_handling.rb`): Request/notification processing
- **Transport Layer** (`lib/vector_mcp/transport/`): Communication protocols
  - `HttpStream` (`lib/vector_mcp/transport/http_stream.rb`): **RECOMMENDED** - MCP-compliant streamable HTTP transport
    - `HttpStream::SessionManager`: Thread-safe session lifecycle management
    - `HttpStream::EventStore`: Event storage for resumable connections
    - `HttpStream::StreamHandler`: Server-Sent Events streaming with resumability
- **Handlers** (`lib/vector_mcp/handlers/`): Request processing logic with authorization checks
- **Sampling** (`lib/vector_mcp/sampling/`): Server-initiated LLM requests with streaming support
- **Definitions** (`lib/vector_mcp/definitions.rb`): Tool, Resource, and Prompt definitions with image support
- **Security** (`lib/vector_mcp/security/`): Authentication and authorization framework
- **Logging** (`lib/vector_mcp/logging/`): Structured logging system with observability features
- **Image Processing** (`lib/vector_mcp/image_util.rb`): Image handling and MCP format conversion
- **Middleware** (`lib/vector_mcp/middleware/`): Pluggable hook system for custom behavior

### Key Features

- **Tools**: Custom functions that LLMs can invoke with optional security policies and image support
- **Resources**: Data sources for LLM consumption with access control and image resource helpers
- **Prompts**: Structured prompt templates with image argument support
- **Roots**: Filesystem boundaries for security and workspace context
- **Sampling**: LLM completion requests with streaming, tool calls, and image support
- **Security**: Comprehensive authentication and authorization system
- **Logging**: Component-based structured logging with multiple outputs and formats
- **Image Processing**: Full image handling pipeline with format detection and validation
- **Middleware**: Pluggable hooks for custom behavior around all MCP operations

### Request Flow

**HTTP Stream Transport (Recommended):**
1. Client sends JSON-RPC requests (`POST /mcp`) with `Mcp-Session-Id` header

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sergiobayona/vector_mcp](https://github.com/sergiobayona/vector_mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
