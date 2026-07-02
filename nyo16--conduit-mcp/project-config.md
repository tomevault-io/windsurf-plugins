---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ConduitMCP is an Elixir library implementing the [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) specification (version 2025-06-18). It lets developers build MCP servers that expose tools, resources, and prompts to LLM applications like Claude Desktop and VS Code. The library is published on Hex as `conduit_mcp`.

## Build & Test Commands

```bash
mix deps.get              # Install dependencies
mix compile               # Compile
mix test                  # Run all tests (~526 tests)
mix test test/conduit_mcp/dsl_test.exs           # Run a single test file
mix test test/conduit_mcp/dsl_test.exs:42        # Run a specific test line
mix format                # Format code
mix credo                 # Lint
mix dialyzer              # Static analysis (slow first run)
mix docs                  # Generate docs
mix bench                 # Run all benchmarks
mix bench validation      # Run a specific benchmark
mix bench --list          # List available benchmarks
MIX_ENV=test mix coveralls.html  # Coverage report
```

Requires Elixir ~> 1.18. No config directory exists; validation config is set via `Application.put_env(:conduit_mcp, :validation, [...])` in tests or application config.

## Architecture

MCP server modules are **stateless** — just compiled functions. Each HTTP request runs in its own Bandit process for maximum concurrency. No supervised processes — rate limiting uses a user-supplied Hammer module.

### Request Flow

```
HTTP Request → Transport (StreamableHTTP/SSE) → Auth Plug → Rate Limit Plug → Handler → Server Module callbacks
```

### Key Modules

- **`ConduitMcp.Server`** — Behaviour with 6 optional callbacks (`handle_list_tools/1`, `handle_call_tool/3`, `handle_list_resources/1`, `handle_read_resource/2`, `handle_list_prompts/1`, `handle_get_prompt/3`). `use ConduitMcp.Server` enables DSL mode; `use ConduitMcp.Server, dsl: false` gives manual control.

- **`ConduitMcp.Endpoint`** — Third mode: aggregates `ConduitMcp.Component` modules into a server. Uses `@before_compile` to generate all `ConduitMcp.Server` callbacks by dispatching to registered components. Carries declarative rate_limit/message_rate_limit/auth config via `__endpoint_config__/0`, auto-extracted by transports.

- **`ConduitMcp.Component`** — Behaviour for individual tool/resource/prompt modules. Each component has `execute/2` callback receiving atom-keyed params and `Plug.Conn`. Uses `ConduitMcp.Component.Schema` for field definitions. `@before_compile` generates introspection functions (`__component_type__/0`, `__component_name__/0`, `__component_schema__/0`, `__validation_schema__/0`). Tool components support `ui:` option for MCP Apps.

- **`ConduitMcp.DSL`** — Compile-time macro system. Accumulates `@mcp_tools`, `@mcp_prompts`, `@mcp_resources` module attributes via `tool`, `prompt`, `resource`, `app` macros. The `@before_compile` hook generates all callback implementations and validation schemas. Tools support `meta/1` (arbitrary `_meta` metadata), `ui/1` (shortcut for `_meta.ui.resourceUri`), and `app/2` (registers both a tool with UI and its `ui://` resource).

- **`ConduitMcp.DSL.SchemaBuilder`** — Dual schema generation: JSON Schema (for MCP client introspection) and NimbleOptions schema (for server-side runtime validation). Both are generated from the same DSL param definitions at compile time. Reused by `ConduitMcp.Component` for schema generation.

- **`ConduitMcp.DSL.Helpers`** — Response helper macros (`text/1`, `json/1`, `raw/1`, `error/1`, `image/1`, `audio/2`, `raw_resource/2`, `system/1`, `user/1`, `assistant/1`). These are macros, not functions — they expand to `{:ok, %{...}}` or `{:error, %{...}}` tuples. Imported automatically in DSL and Component modes. `raw_resource/2` returns resource content with a specified MIME type.

- **`ConduitMcp.Errors`** — Centralized JSON-RPC 2.0 and MCP error code constants. Used across all modules instead of hardcoded integers. `ConduitMcp.Protocol` delegates to this module.

- **`ConduitMcp.Handler`** — Routes JSON-RPC 2.0 requests to server module callbacks. Handles `initialize`, `ping`, `tools/*`, `resources/*`, `prompts/*`. Uses `__capabilities__/0` for selective capability advertisement when available. Emits telemetry events.

- **`ConduitMcp.Protocol`** — JSON-RPC 2.0 message construction and validation. Error code functions delegate to `ConduitMcp.Errors`.

- **`ConduitMcp.Validation`** — Runtime parameter validation using NimbleOptions. Validates tool params and prompt args. Supports type coercion (string→integer, string→boolean), custom constraints (min/max, min_length/max_length, enum), and custom validator functions.

- **`ConduitMcp.Validation.SchemaConverter`** — Converts DSL param definitions to NimbleOptions schemas. Custom constraint markers (prefixed with `__`, e.g., `__enum_values__`, `__min_value__`) are stored alongside NimbleOptions options and stripped before NimbleOptions validation.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nyo16/conduit_mcp](https://github.com/nyo16/conduit_mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
