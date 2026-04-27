---
trigger: always_on
description: This document describes the `pls` codebase for agentic coding agents working in this repository.
---

# Agent Guide

This document describes the `pls` codebase for agentic coding agents working in this repository.

## Build, Test, and Run

```sh
# Build
zig build

# Run all tests
zig build test

# Run a single test by name filter
zig build test -- --filter "isDestructive detects kill"

# Run tests in one file directly (faster, no full build graph)
zig test src/agent.zig
zig test src/config.zig
zig test src/tools/shell.zig
zig test src/llm/provider.zig
zig test src/llm/json_helpers.zig

# Run the binary
zig build run -- 'your task here'
zig build run -- --dry-run 'your task here'

# Format all source files (authoritative)
zig fmt src/
```

The project uses **Zig 0.15.2**. There is no separate linter; `zig fmt` is the only formatter.

---

## Project Structure

```
src/
  main.zig            CLI entry point, arg parsing, runTask()
  agent.zig           Agent loop, tool dispatch, system prompt building
  config.zig          Config struct, TOML parser, env var overrides
  config_editor.zig   Interactive config editor (terminal UI)
  init.zig            First-run setup wizard
  tty.zig             TTY helpers (raw mode, echo control)
  llm/
    provider.zig      Shared types: Message, ToolCall, ContentBlock, Tool
    anthropic.zig     Anthropic Claude API client
    openai.zig        OpenAI API client
    gemini.zig        Google Gemini API client
    ollama.zig        Ollama local API client
    http_client.zig   Thin std.http.Client POST wrapper
    json_helpers.zig  Per-provider JSON serialisers and tests
  tools/
    shell.zig         Shell command execution, ShellResult
    confirm.zig       ask() y/N prompt, askUser() numbered options
```

---

## Architecture

### Agent Loop (`src/agent.zig`)

Turn-based loop capped at `max_turns` (default 20):

1. Call LLM with full message history
2. Print LLM text blocks as thinking — dim grey `~ text`
3. Handle `ask_user` tool calls immediately (clarification before acting)
4. Collect all `run_shell` tool calls into a batch
5. Display every command in the batch (`  $ cmd`)
6. Ask `Execute? [y/N]` once for the whole batch (respects `confirm_mode`)
7. If declined: stop immediately, return
8. Execute each command in order, print output to stderr
9. Feed all results back to LLM and repeat

### Tools

| Tool | Purpose | Dispatch |
|---|---|---|
| `run_shell` | Execute shell commands | Batched — confirmed once per LLM response |
| `ask_user` | Clarify ambiguous user intent | Immediate — answered before shell batch |

The old `confirm` tool no longer exists. Execution confirmation is fully agent-driven via `confirm_mode`.

### Confirm Modes

- `all` — prompt before every batch
- `destructive` — prompt only if any command in the batch matches `isDestructive()`
- `none` — execute without prompting

### System Prompt

Built dynamically at `Agent.init()` by `buildSystemPrompt()`. Appends a live `Environment:` section (OS, kernel, arch, hostname, user, shell, cwd, home) to the static `SYSTEM_PROMPT_BASE` string.

### Message Format

All providers share `provider.Message` internally:
- `role`: `user | assistant | tool`
- `content`: slice of `ContentBlock` — `.text` or `.tool_call`
- `tool_call_id`: set on tool-result messages to link back to the call

Wire-format differences:
- **Anthropic**: tool calls → `tool_use` blocks; results → `tool_result` in user messages
- **OpenAI/Ollama**: tool calls → `tool_calls` array on assistant message; results → `tool` role messages
- **Gemini**: tool calls → `functionCall` parts; results → `functionResponse` parts in user messages; uses function name as tool call ID; system prompt goes in `system_instruction`

---

## Code Style

### Imports

```zig
const std = @import("std");
const Allocator = std.mem.Allocator;           // always alias this
const builtin = @import("builtin");            // only when compile-time OS/arch needed
const provider = @import("provider.zig");      // relative paths for internal modules
```

### Naming

| Kind | Convention | Example |
|---|---|---|
| Types, structs, enums | `PascalCase` | `ConfirmMode`, `ShellResult` |
| Functions | `camelCase` | `buildRequestBody`, `isDestructive` |
| Variables, fields | `snake_case` | `confirm_mode`, `api_key` |
| File-level constants | `SCREAMING_SNAKE_CASE` | `SYSTEM_PROMPT_BASE`, `API_URL` |
| Test names | Sentence string | `"isDestructive detects kill commands"` |

### Memory Management

- Every allocating function takes `allocator: Allocator` as its first parameter.
- Callers own returned slices and must free them.
- Use `defer allocator.free(x)` immediately after allocation.
- Use `errdefer buf.deinit(allocator)` on `ArrayList` buffers before `toOwnedSlice`.
- Structs that own heap memory expose `pub fn deinit(self: *Self, allocator: Allocator) void`.

Typical string-building pattern:
```zig
var buf: std.ArrayList(u8) = .empty;
errdefer buf.deinit(allocator);
const w = buf.writer(allocator);
try w.print("...", .{});
return buf.toOwnedSlice(allocator);
```

### Error Handling

- Return errors with `!ReturnType`; let errors propagate with `try`.
- Use named errors (`error.JsonParseError`, `error.ApiError`, `error.HttpError`).
- Catch only to remap to a typed error or to handle locally:
  ```zig

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [colus001/pls](https://github.com/colus001/pls) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
