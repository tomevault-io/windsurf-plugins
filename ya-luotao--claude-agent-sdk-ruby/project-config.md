---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Unofficial, community-maintained Ruby SDK for Claude Agent (gem: `claude-agent-sdk`). Wraps the Claude Code CLI as a subprocess, communicating via stream-JSON over stdin/stdout. Requires Ruby 3.2+ and Claude Code CLI 2.0.0+.

Runtime dependencies: `async` (~2.0) for concurrency, `mcp` (~0.4) for MCP protocol compliance.

## Common Commands

```bash
bundle install                              # Install dependencies
bundle exec rspec                           # Run all unit tests (integration tests excluded by default)
bundle exec rspec spec/unit/types_spec.rb   # Run a single spec file
bundle exec rspec spec/unit/types_spec.rb:42  # Run a single test by line number
RUN_INTEGRATION=1 bundle exec rspec         # Include integration tests (requires Claude Code CLI)
bundle exec rubocop                         # Run linter
bundle exec rake                            # Run default task (spec + rubocop)
bundle exec rake build                      # Build the gem
```

## Architecture

### Layered Design

```
User code
  ├── ClaudeAgentSDK.query()     ← One-shot/streaming, defined in lib/claude_agent_sdk.rb
  └── ClaudeAgentSDK::Client     ← Bidirectional sessions, defined in lib/claude_agent_sdk.rb
        │
        ▼
      Query (lib/claude_agent_sdk/query.rb)
        - Bidirectional control protocol (control_request / control_response routing)
        - Hook callback dispatch with typed inputs (PreToolUseHookInput, etc.)
        - Permission callback handling (can_use_tool)
        - SDK MCP server request routing (tools/call, resources/read, prompts/get)
        - Message queue (Async::Queue) separating control messages from SDK messages
        │
        ▼
      SubprocessCLITransport (lib/claude_agent_sdk/subprocess_cli_transport.rb)
        - Spawns `claude` CLI via Open3.popen3
        - Builds CLI command from ClaudeAgentOptions
        - Reads stdout as newline-delimited JSON, writes stdin for streaming mode
        - Stderr handling in a separate Thread
        │
        ▼
      Claude Code CLI (external Node.js process)
```

### Two API Entry Points

- **`query()`** — Simple function interface. Creates a `SubprocessCLITransport` directly, reads messages via `transport.read_messages`, parses with `MessageParser`. No control protocol. Good for one-shot queries and streaming input via Enumerators.

- **`Client`** — Full-featured bidirectional sessions. Accepts optional `transport_class` (defaults to `SubprocessCLITransport`) and `transport_args` for custom transports. Creates transport, instantiates a `Query` handler that runs `read_messages` in an async task, routes control messages internally, and exposes SDK messages via `Async::Queue`. Supports hooks, permission callbacks, SDK MCP servers, interrupt, model switching, and file rewind.

### SDK MCP Servers

Custom tools run in-process (no subprocess). The flow:
1. User defines tools via `ClaudeAgentSDK.create_tool` → returns `SdkMcpTool`
2. `create_sdk_mcp_server` wraps tools in `SdkMcpServer`, which creates dynamic `MCP::Tool` subclasses and delegates to the official `MCP::Server`
3. The server config hash (`{ type: 'sdk', instance: server }`) is stored in `ClaudeAgentOptions.mcp_servers`
4. `Client.connect` extracts SDK server instances and passes them to `Query`
5. `Query.handle_control_request` dispatches `mcp_message` subtypes to the server's `list_tools` / `call_tool` / etc.

### Message Flow

CLI outputs newline-delimited JSON. `SubprocessCLITransport.read_messages` parses it and yields raw hashes. In `Client` mode, `Query.read_messages` intercepts `control_response` and `control_request` types, putting regular messages on `@message_queue`. `MessageParser.parse` converts raw hashes into typed objects (`UserMessage`, `AssistantMessage`, `SystemMessage`, `ResultMessage`, `StreamEvent`).

### Control Protocol

Only active in streaming/Client mode. Uses `Async::Condition` for request-response coordination:
- **Outbound requests** (SDK → CLI): `send_control_request` writes JSON, waits on condition, returns response
- **Inbound requests** (CLI → SDK): `handle_control_request` dispatches to `can_use_tool`, `hook_callback`, or `mcp_message` handlers, writes response back

### Observer / Instrumentation

Optional observability layer for tracing agent sessions (e.g., Langfuse via OpenTelemetry).

- **`Observer`** module (`lib/claude_agent_sdk/observer.rb`) — base interface with no-op defaults: `on_user_prompt(prompt)`, `on_message(message)`, `on_error(error)`, `on_close`
- **`OTelObserver`** (`lib/claude_agent_sdk/instrumentation/otel.rb`) — emits OTel spans with `gen_ai.*` + OpenInference semantic conventions. Lazy-requires `opentelemetry-api`; not loaded unless user explicitly `require 'claude_agent_sdk/instrumentation'`
- Observers registered via `ClaudeAgentOptions.new(observers: [...])`. Supports callable factories (lambdas) for thread-safe global defaults in Rails
- `resolve_observers` materializes callables into fresh instances per query/session; `notify_observers` calls methods with `rescue StandardError` so observers never crash the pipeline

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ya-luotao/claude-agent-sdk-ruby](https://github.com/ya-luotao/claude-agent-sdk-ruby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
