---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an MCP (Model Context Protocol) server that bridges MCP clients with DAP (Debug Adapter Protocol) debuggers. It exposes debugging capabilities as MCP tools, allowing AI assistants to programmatically control debuggers. Supports Delve (Go) and GDB (C/C++ via native DAP).

## Architecture

### Core Components

**main.go**: MCP server initialization
- Creates the MCP server using the `go-sdk`
- Registers all debugging tools via `registerTools()`
- Registers workflow prompts via `registerPrompts()`
- Exposes stdio transport

**prompts.go**: MCP prompt implementations
- 4 prompt handlers for guided debugging workflows (source, attach, core dump, binary)
- Registered via `server.AddPrompt()` — no session state, always available
- Each prompt returns a `GetPromptResult` with step-by-step tool invocation guidance

**tools.go**: MCP tool implementations (~1200 lines)
- All MCP tools are methods on `debuggerSession` struct
- Each tool method signature: `func (ds *debuggerSession) toolName(ctx context.Context, _ *mcp.ServerSession, params *mcp.CallToolParamsFor[ParamsType]) (*mcp.CallToolResultFor[any], error)`
- Tools send DAP requests via `ds.client` and read/parse responses
- `readAndValidateResponse(client, requestSeq, errorPrefix)` matches responses by `request_seq`, skipping unrelated responses
- `readTypedResponse[T](client, requestSeq)` matches typed responses by `request_seq`, skipping unrelated responses
- Tools that wait for stopped/terminated events loop on `ReadMessage()` until they receive the expected event

**dap.go**: DAP client implementation
- `DAPClient` manages TCP or stdio connection to DAP server
- Wraps `github.com/google/go-dap` protocol messages
- All DAP request methods return `(int, error)` where `int` is the request sequence number
- Sequence numbers are used to match responses to requests via `request_seq` field

**backend.go**: Debugger backend abstraction
- `DebuggerBackend` interface abstracts debugger-specific behavior (spawning, launch args, transport)
- `delveBackend`: Spawns `dlv dap`, uses TCP transport
- `gdbBackend`: Spawns `gdb -i dap`, uses stdio transport

**flexint.go**: Flexible integer parsing
- `FlexInt` type handles JSON values that may be integers or string-encoded integers
- Used in tool parameter structs where MCP clients may send numbers as strings

**debuggerSession**: Shared state (tools.go)
- `cmd`: The debugger adapter process
- `client`: DAP client connection
- `server`: MCP server reference (for dynamic tool registration)
- `backend`: Debugger-specific backend (delve, gdb)
- `capabilities`: DAP capabilities reported by the adapter
- `launchMode`, `programPath`, `programArgs`, `coreFilePath`: Session config
- `stoppedThreadID`, `lastFrameID`: State from last stop event
- All tool methods operate on this shared session

### Key Patterns

1. **Tool Parameter Structs**: Each tool has a corresponding `*Params` struct with JSON/MCP tags
2. **DAP Message Handling**: Response reading matches by `request_seq` (from the DAP protocol), skipping unrelated responses from other requests. go-dap decodes all failed responses as `*dap.ErrorResponse`, so matching by Go type alone is insufficient
3. **Event vs Response**: Some operations (continue, step, etc.) wait for `StoppedEvent` or `TerminatedEvent` rather than just response messages
4. **Error Propagation**: DAP response `Success` field is checked; error messages from `response.Message` are wrapped in Go errors
5. **Capability-Gated Tools**: `set-variable`, `disassemble`, and `restart` are only registered when the DAP adapter reports support
6. **Dynamic Tool Registration**: Only `debug` is registered initially; session tools replace it after a session starts, then are removed when the session stops
7. **Serialized DAP Access**: A mutex on `debuggerSession` serializes all tool calls, preventing concurrent reads from the single DAP connection

## Development Commands

### Build
```bash
go build -o bin/mcp-dap-server
```

### Run Tests
```bash
# Run all tests
go test -v

# Run a specific test
go test -v -run TestBasic

# Run with race detector
go test -race -v

# Run with coverage
go test -v -coverprofile=coverage.out && go tool cover -func=coverage.out | grep total
```

### Run the Server
```bash
# Default (port 8080)
./bin/mcp-dap-server

# Or via go run
go run .
```

### Test Program Compilation
Test programs are compiled with debugging symbols during test execution:
```bash
go build -gcflags=all=-N -l -o testdata/go/helloworld/debugprog testdata/go/helloworld/main.go
```

## MCP Tools (Current API)

Tools are dynamically registered. Before a debug session, only `debug` is available. After starting a session, the following tools are registered:

| Tool | Description |
|------|-------------|
| `debug` | Start a complete debug session (modes: source, binary, core, attach) |
| `stop` | End the debugging session |
| `breakpoint` | Set a breakpoint (file+line or function name) |
| `clear-breakpoints` | Remove breakpoints (by file or all) |
| `continue` | Continue execution (with optional run-to-cursor) |
| `step` | Step over/in/out |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [go-delve/mcp-dap-server](https://github.com/go-delve/mcp-dap-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
