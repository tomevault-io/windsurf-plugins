---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents when working with code in this repository.

## Project Overview

**Yosh** is an LLM-enabled shell - a custom build of GNU Bash 5.2.32 with GNU Readline 8.2.13, compiled using the Fil-C memory-safe compiler toolchain. The entire stack (bash, readline, libcurl, openssl, zlib, libc) is compiled with Fil-C for memory safety.

The key feature is the "yo" command: type `yo <natural language>` and the shell calls an LLM (Anthropic Claude, OpenAI, or Kimi) to either generate a shell command or answer a question.

## Build System

- **Full build**: `./build.sh` — configures and builds readline + bash from scratch. Requires Fil-C at `/opt/fil`.
- **Incremental build**: `./build_incremental.sh` — faster rebuilds when making code changes.
- **Output**: The resulting shell binary ends up at `../fil-c-2/pizfix/bin/yosh`

## Project Structure

```
yosh/
├── readline-8.2.13/    # GNU Readline with "yo" LLM integration
│   ├── yo.c            # All LLM code: API, session memory, PTY proxy, scrollback, continuation
│   ├── yo.h            # Public API for yo feature
│   └── cJSON.[ch]      # Embedded JSON parser (MIT licensed)
├── bash-5.2.32/        # Yosh shell (bash fork)
│   ├── shell.c         # Main init (readline before job control - critical!)
│   ├── bashline.c      # Calls rl_yo_enable() with system prompt
│   └── version.c       # "Fil's yosh" branding
├── build.sh            # Full build script
└── build_incremental.sh
```

## The "yo" Feature

### Architecture

The yo feature is an **opt-in readline extension** (like history). Readline provides `yo.c` with all LLM logic; bash provides the system prompt via `rl_yo_enable(prompt)`.

**Multi-provider support**: yo supports Anthropic (Claude), OpenAI, and Kimi APIs. The provider is selected via `~/.yoconf`. The architecture keeps provider-specific code separated:
- Message building uses provider-aware helpers (`yo_msg_add_tool_use`, `yo_msg_add_tool_result`) that produce native JSON for each provider from C parameters
- HTTP infrastructure is shared (`yo_http_post`) with curl multi-handle and Ctrl-C cancellation
- Request building is per-provider (`yo_build_anthropic_request`, `yo_build_openai_request`, `yo_build_kimi_request`)
- Response parsing is per-provider (`yo_parse_anthropic_response`, `yo_parse_openai_response`, `yo_parse_kimi_response`), all producing a normalized internal tool_use format

**API details**:
- Anthropic uses the Messages API (`/v1/messages`) with server-side tools (`web_search_20250305`, `web_fetch_20250910`)
- OpenAI uses the Responses API (`/v1/responses`, NOT Chat Completions) with `{"type":"web_search"}` tool for web search
- Kimi uses the Chat Completions API (`/v1/chat/completions`)
- The Responses API always returns `"error": null` on success — error checking must use `cJSON_IsNull()` to avoid false positives
- OpenAI Responses API uses flat items in `input[]` (`{"type":"function_call",...}`, `{"type":"function_call_output",...}`) rather than role-based messages for tool interactions
- OpenAI Responses API uses `"instructions"` for system prompt (not a system message in the input array), `"input"` instead of `"messages"`, `"max_output_tokens"` instead of `"max_completion_tokens"`, and `"output[]"` instead of `"choices[].message"`
- Tool definitions are built separately per provider. OpenAI tools are stricter (e.g., `command.pending` is required and descriptions strongly bias toward command/tool use).
- OpenAI scrollback is sanitized (ANSI/escape sequences stripped) before sending it to the model; Anthropic receives raw scrollback.

Response types from the LLM:
- **command** — `{"type":"command","command":"...","explanation":"..."}` — prefills the command in the prompt for the user to review/edit/execute.
- **command with pending** — same but with `"pending":true` — triggers multi-step continuation (see below).
- **chat** — `{"type":"chat","response":"..."}` — prints the response, returns to fresh prompt.
- **scrollback** — `{"type":"scrollback","lines":N}` — yo fetches terminal scrollback and makes a follow-up API call.

### Configuration

**Config file (`~/.yoconf`)** — Read fresh on each yo request. Supports `#` comments. All directives are optional:
```
# Provider: "anthropic", "openai", or "kimi"
# The provider determines the API style used. When using base_url, the provider
# field selects which API format to use:
#   - anthropic = Anthropic Messages API style
#   - openai = OpenAI Responses API style
#   - kimi = OpenAI Chat Completions API style
provider anthropic

# Model name (provider-specific)
model claude-sonnet-4-5-20250929

# API key (optional if using a key file instead)
key sk-ant-api03-...

# Base URL for API requests (optional)
# When set, overrides the default API URL. The provider field determines the
# API style. The endpoint path is appended automatically:
#   - anthropic: /messages
#   - openai: /responses
#   - kimi: /chat/completions
# Example: base_url https://api.moonshot.ai/v1/
# base_url https://your-custom-endpoint.com/v1/

# Chat display prefix/reset (C-style escapes supported, optional quoting with " or ')
# chat_prefix \033[3;36m
# chat_reset \033[0m
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pizlonator/yosh](https://github.com/pizlonator/yosh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
