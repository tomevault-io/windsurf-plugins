---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

WailsChat is a desktop AI chat application built with **Wails v2** (Go backend + Vue 3 frontend). It connects to OpenAI-compatible LLM APIs with SSE streaming, stores data locally in SQLite, supports multiple providers/models, and includes **multimodal image input support** for vision-capable models like GPT-4 Vision. Features **performance statistics tracking**, **custom CSS styling**, **message retry/cancel functionality**, **configurable keyboard shortcuts**, **background image customization**, **resizable sidebar**, and **session drag-and-drop reordering**.

**Key Features**:
- **Built-in Tools (Read/Write/Execute)**: AI models can read local files, write files, and execute shell commands directly
- **MCP (Model Context Protocol) Tool Calling**: AI models can use external tools via standardized MCP protocol
- **LaTeX Mathematical Rendering**: High-quality math formula rendering using KaTeX
- **Mermaid Diagram Support**: Render Mermaid diagrams in code blocks
- **Enhanced Markdown Processing**: Improved bold rendering for Unicode punctuation

## Commands

```bash
# Development (starts Go backend + Vite dev server with hot reload)
wails dev

# Production build (outputs to build/bin/wailschat)
wails build

# Frontend only (no Go backend)
cd frontend && npm run dev

# Frontend type check + build
cd frontend && npm run build

# Go dependency management
go mod tidy
```

Dev server runs at http://localhost:34115 with Wails Inspector for debugging Go bindings.

## Architecture

### Backend → Frontend Communication

- **RPC calls**: Frontend calls Go methods via auto-generated Wails bindings in `frontend/wailsjs/go/main/`. These are regenerated on `wails dev`/`wails build` — **never edit them manually**.
- **Events (streaming)**: Backend emits events via `runtime.EventsEmit()`:
  - `message_chunk` — SSE content delta for real-time rendering
  - `message_done` — Streaming completed successfully
  - `message_error` — Error during streaming
  - `message_stats` — Performance statistics (tokens, timing, speed)
  - `session_renamed` — Auto-generated title update
  - `mcp_tool_call_start` — MCP tool call started
  - `mcp_tool_result` — MCP tool call result
  - `mcp_server_status` — MCP server connection status change
- Frontend listens via `EventsOn()` in Pinia stores.

### Backend Structure

- `main.go` — Entry point. Initializes Wails window (1200x900, min 800x600), embeds frontend assets and logo icon, binds `App` struct.
- `app.go` — `App` struct with all exported methods that become Wails RPC endpoints. Includes:
  - Provider/Session/Settings CRUD operations
  - `SendMessage()` — Saves user message, starts streaming goroutine with MCP tool calling support
  - `RetryMessage()` — Deletes message and all after it, re-streams
  - `RetryFromUserMessage()` — Deletes from a user message onwards, re-saves it, and re-streams
  - `CancelMessage()` — Cancels in-flight streaming via context
  - `ClearSessionMessages()` — Deletes all messages in a session (clears context)
  - `ReorderSessions()` — Persists new display order after drag-and-drop
  - `GetSystemFonts()` — Returns system font list (cross-platform)
  - `GetDefaultStyles()` — Returns built-in CSS stylesheet
  - `GetModels()` — Fetches available models from API endpoint
  - `OpenFileDialog()` — Native file picker for background image selection
  - `ReadImageAsBase64()` — Reads local image file as data URI
  - **MCP Server Management**: `MCPServerList()`, `MCPServerCreate()`, `MCPServerUpdate()`, `MCPServerDelete()`, `MCPServerTest()`, `MCPServerConnect()`, `MCPServerDisconnect()`, `MCPServerGetStatus()`, `MCPServerGetAllStatuses()`, `MCPServerGetTools()`
  - **MCP Tool Calling**: `GetEnabledMCPTools()`, `CallMCP_tool()`
- `window_state.go` — Window position/size/maximized state persistence across sessions.
- `internal/db/` — SQLite initialization, schema creation, migrations (V1→V15). DB stored at platform config directory (`%LOCALAPPDATA%/wailschat/wailschat_data.db` on Windows). Contains `DefaultStyles()` with built-in CSS.
- `internal/llm/` — HTTP client for OpenAI-compatible APIs:
  - `client.go` — `StreamChat()` (SSE with performance stats and tool calling support), `Chat()` (non-streaming for title generation), `TestConnection()`, `GetModels()`
  - `sse_parser.go` — Parses `data: {...}` lines, extracts content, usage stats, and tool calls
- `internal/model/` — Data models:
  - `Provider`, `Session`, `Message` (with `images`, `stats`, `tool_calls`, `tool_results` fields)
  - `ChatMessage` (multimodal content support), `ChatCompletionRequest`
  - `PerformanceStats` — Input/output tokens, first token time, total time, speed (tokens/sec)
  - `UsageInfo`, `StreamOptions`, `ContentText`, `ContentImage`
  - **MCP Models**: `MCPServer`, `MCPTool`, `MCPServerTestResult`, `ToolCall`, `ToolCallResult`, `FunctionCall`, `FunctionDef`, `Tool`
- `internal/mcp/` — MCP protocol implementation:
  - `client.go` — MCP client manager with stdio transport support, connection pooling, tool caching, and tool execution

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jacksalad/wailschat](https://github.com/jacksalad/wailschat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
