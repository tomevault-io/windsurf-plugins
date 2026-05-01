---
trigger: always_on
description: and anthropi# Sofos - AI Coding Assistant Project Context
---

and anthropi# Sofos - AI Coding Assistant Project Context

## Project Overview

Sofos is a terminal-based AI coding assistant powered by Anthropic's Claude API. It's built in Rust for maximum performance and security. The assistant can read/write files, search code, execute bash commands, and search the web - within the workspace by default, with interactive permission prompts for external paths.

**Core Philosophy:**
- Security first: Workspace-sandboxed by default, with interactive user-approved access to external paths via three independent scopes (Read, Write, Bash)
- Fast and efficient: Native Rust implementation with optional ultra-fast editing via Morph API
- Developer-friendly: Interactive REPL with session persistence and custom instructions
- Transparent: All tool executions are visible to the user

## Architecture

### Key Design Decisions

1. **Dual Session Storage Format** (src/session/history.rs)
   - `api_messages`: Anthropic API format for continuing conversations
   - `display_messages`: UI-friendly format for showing conversation history
   - This separation ensures Claude sees proper API format while users see original UI

2. **Tool Calling Pattern** (src/repl/mod.rs)
   - Assistant returns content blocks (text + tool_use)
   - REPL executes tools and collects results
   - Results sent back as user message with tool_result blocks
   - **Loop-based handling** allows Claude to use multiple tools in sequence iteratively

3. **Two-Level Instructions** (src/session/history.rs)
   - `AGENTS.md`: Project-level, version controlled
   - `.sofos/instructions.md`: Personal, gitignored
   - Both appended to system prompt at startup

4. **Sandboxing Strategy** (src/tools/filesystem.rs, src/tools/bashexec.rs)
   - All paths validated before operations
   - Parent directory traversal blocked (`..`)
   - Absolute paths rejected
   - Symlinks checked to prevent escape
   - Bash commands filtered through blocklist

## Code Organization

### Directory Structure

```
src/
├── main.rs              # Entry point
├── cli.rs               # CLI argument parsing
├── error.rs             # Error types
├── error_ext.rs         # Error extensions
├── config.rs            # Configuration (SofosConfig, ModelConfig)
│
├── api/                 # API clients
│   ├── anthropic.rs     # Claude API client
│   ├── openai.rs        # OpenAI API client
│   ├── morph.rs         # Morph Apply API client
│   ├── types.rs         # Message types and serialization
│   └── utils.rs         # API utilities
│
├── mcp/                 # MCP (Model Context Protocol) integration
│   ├── mod.rs           # MCP module exports
│   ├── config.rs        # MCP server configuration loading
│   ├── protocol.rs      # MCP protocol types (JSON-RPC, tools)
│   ├── client.rs        # MCP client implementations (stdio, HTTP)
│   └── manager.rs       # MCP server connection management
│
├── repl/                # REPL components
│   ├── mod.rs           # Main REPL loop and Repl struct
│   ├── clipboard_edit_mode.rs # Custom EditMode wrapping Emacs to intercept Ctrl+V
│   ├── conversation.rs  # Message history management
│   ├── prompt.rs        # Prompt rendering
│   ├── request_builder.rs   # API request construction
│   └── response_handler.rs  # Response processing
│
├── session/             # Session management
│   ├── history.rs       # Session persistence + custom instructions
│   ├── state.rs         # Runtime session state
│   └── selector.rs      # Session selection TUI
│
├── clipboard.rs         # Clipboard image paste (Ctrl+V) with numbered markers
│
├── tools/               # Tool implementations
│   ├── filesystem.rs    # File operations (read, write, list, etc)
│   ├── bashexec.rs      # Sandboxed bash execution
│   ├── codesearch.rs    # Ripgrep integration
│   ├── image.rs         # Image handling (local paths, URLs)
│   ├── permissions.rs   # 3-tier command permission system
│   ├── tool_name.rs     # Type-safe tool name enum
│   ├── types.rs         # Tool definitions for API
│   └── utils.rs         # Tool utilities (confirmations, HTML-to-text)
│
├── ui/                  # UI components
│   ├── mod.rs           # Main UI utilities and display logic
│   ├── syntax.rs        # Markdown/code syntax highlighting
│   └── diff.rs          # Contextual diff generation and display
│
└── commands/            # Built-in commands
    └── builtin.rs       # Command implementations
```

### Key Files

**src/api/types.rs**
- Defines Message, ContentBlock, and MessageContentBlock enums
- Handles serialization/deserialization for Anthropic API
- Supports both regular and server-side tools (like web_search)

**src/ui/diff.rs**
- Generate contextual diffs with syntax highlighting and line numbers
- Uses `similar` crate for diffing, `syntect` for syntax coloring
- Dark backgrounds (#5e0000 deletions, #00005f additions) with syntax-colored code
- Context lines (default: 2) show unchanged code around changes
- Used by edit_file, write_file, and morph_edit_file tools

**src/session/history.rs**
- SessionMetadata: Preview and timestamps for session list
- Session: Dual storage (api_messages + display_messages)
- DisplayMessage: Enum for user messages, assistant responses, and tool executions

**src/repl/mod.rs**
- Main event loop (run method)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexylon/sofos-code](https://github.com/alexylon/sofos-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
