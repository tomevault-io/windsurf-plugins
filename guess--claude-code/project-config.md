---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the ClaudeCode Elixir SDK - an idiomatic Elixir interface to the Claude Code CLI. The SDK spawns the `claude` command as a subprocess and communicates via streaming JSON over stdout.

## Common Development Commands

### Quality Checks
```bash
mix quality          # Run all code quality checks (compile, format, credo, dialyzer)
mix format           # Format code with Styler
mix credo --strict   # Run Credo analysis
mix dialyzer         # Run Dialyzer type checking
```

### Testing
```bash
mix test                        # Run all tests
mix test test/path/to_test.exs # Run specific test file
mix test.all                    # Run tests with coverage report
mix coveralls.html              # Generate HTML coverage report

# Distributed tests (Adapter.Node) require a named BEAM node:
elixir --sname test -S mix test --include distributed
```

### Development
```bash
mix deps.get              # Install dependencies
mix claude_code.install   # Install CLI binary to priv/bin/
mix claude_code.uninstall # Remove bundled CLI binary
mix claude_code.path      # Print resolved CLI binary path
iex -S mix                # Start interactive shell with project loaded
mix docs                  # Generate documentation
```

## Architecture

The SDK is organized in three layers:

1. **Adapter-agnostic** — Session, Stream, Options (validation), Types, Message/Content structs
2. **CLI protocol** — CLI.Command (flags), CLI.Input (stdin), CLI.Parser (JSON parsing)
3. **Adapters** — Adapter.Port (local CLI via Port), Adapter.Node (distributed via BEAM), Adapter.Test (in-memory stubs)

Key modules:
- **ClaudeCode.Session** - Public API for session operations (lifecycle, runtime config, MCP, introspection, execute)
- **ClaudeCode.Session.Server** - GenServer that manages the adapter subprocess lifecycle
- **ClaudeCode.Adapter.Port** - Port-based adapter that spawns the CLI as a local subprocess
- **ClaudeCode.Adapter.Node** - Distributed adapter that runs Adapter.Port on a remote BEAM node via Erlang distribution
- **ClaudeCode.CLI** - Thin facade: binary resolution + command building (delegates to Resolver + Command)
- **ClaudeCode.CLI.Command** - Converts Elixir options to CLI flags and builds argument lists
- **ClaudeCode.CLI.Parser** - Parses newline-delimited JSON from CLI output into structs
- **ClaudeCode.Adapter.Port.Installer** - CLI binary download and version management
- **ClaudeCode.Plugin** - Plugin management (list, install, uninstall, enable, disable, update)
- **ClaudeCode.Plugin.Marketplace** - Marketplace management (list, add, remove, update)

Key CLI flags used:
- `--input-format stream-json` - Bidirectional streaming mode (reads from stdin)
- `--output-format stream-json` - Get structured JSON output
- `--verbose` - Include all message types

## Current Implementation Status

**26 features implemented** (96% of core functionality) - See `docs/proposals/FEATURE_MATRIX.md`

Core capabilities:
- CLI installer with automatic binary management
- Session management with GenServer
- Synchronous and async query interface
- Streaming support with native Elixir Streams
- Message parsing (System, Assistant, User, Result, StreamEvent)
- Content blocks (Text, ToolUse, ToolResult, Thinking)
- Options API with NimbleOptions validation
- Model selection with fallback model support
- System prompts (override and append)
- Tool control (allowed/disallowed tools, additional directories)
- Permission modes and MCP integration
- Custom agents configuration
- Team settings loading (file path, JSON, or map)
- Session tracking and auto-resume
- Partial message streaming (character-level deltas)
- Stream utilities (text_deltas, thinking_deltas, buffered_text)
- Session forking (via `:fork_session` option with `:resume`)

## Testing Approach

- Unit tests mock the Port for predictable message sequences
- Integration tests use a mock CLI script when the real CLI isn't available
- Property-based testing with StreamData for message parsing
- All new code requires tests

## Important Implementation Notes

- The SDK does NOT make direct API calls - all API communication is handled by the CLI
- Sessions use a persistent CLI subprocess with bidirectional streaming (stdin/stdout)
- The CLI auto-connects on first query and auto-disconnects on session stop
- API keys are passed via environment variables, never in command arguments
- Response content comes from the "result" message, not "assistant" messages
- Spawns the CLI binary directly via `spawn_executable` (no shell wrapper)
- Multi-turn conversations are supported via persistent connection (no subprocess restart between queries)
- Atom safety: use `ClaudeCode.MapUtils.safe_atomize_keys` for map keys from JSON payloads to prevent atom table exhaustion. `String.to_atom` is fine for enum values with finite sets (type, subtype, stop_reason, etc.) since the CLI is a trusted source with a bounded protocol.

## File Structure

- `lib/claude_code/` - Main implementation
  - `session.ex` - Public API for session operations (delegates to Session.Server)
  - `session/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [guess/claude_code](https://github.com/guess/claude_code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
