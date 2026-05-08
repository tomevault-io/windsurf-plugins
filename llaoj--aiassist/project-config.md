---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI Shell Assistant is an intelligent command-line tool for server operations and cloud-native operations. It leverages LLM models to analyze problems, suggest commands, and guide execution through natural language interaction.

Key capabilities:
- Interactive mode with conversation history (up to 10 recursion depth for complex troubleshooting)
- Pipe mode for analyzing command output (supports up to 1.6MB/~13k lines of logs)
- Multi-provider LLM support with automatic fallback on failure
- Safety controls: query commands (green) vs modify commands (red, require double confirmation)
- Internationalization: Chinese and English interfaces

## Build Commands

```bash
# Build for current platform
make build
# Or: ./scripts/build.sh

# Build for all platforms (Linux, macOS, Windows, FreeBSD)
make build-all
# Or: ./scripts/build-all.sh

# Clean build artifacts
make clean
```

## Architecture

### Core Components

**Entry Point**: `cmd/aiassist/main.go`
- Initializes configuration and signal handlers
- Delegates to CLI commands via Cobra framework

**CLI Layer**: `internal/cmd/`
- `root.go`: Main command handling interactive and pipe modes
- `config.go`: Interactive configuration wizard and configuration viewer
- `run.go`: Implements interactive and pipe mode execution flows
- `version.go`: Version information command

**Configuration System**: `internal/config/`
- Supports two modes:
  1. **Local mode**: YAML file at `~/.aiassist/config.yaml`
  2. **Consul mode**: Centralized config from Consul KV store
- When `consul.enabled=true`, all provider/model config comes from Consul
- Local mode: config modified via direct file editing
- Consul mode: read-only from CLI, modified via Consul UI/API/CLI
- Thread-safe with mutex protection for concurrent access

**LLM Manager**: `internal/llm/`
- Manages multiple LLM models across different providers (OpenAI, Alibaba Qwen, custom OpenAI-compatible APIs)
- Automatic fallback: tries models in config file order when one fails
- Tracks model availability (marks unavailable on HTTP errors)
- `manager.go`: Model lifecycle and fallback logic
- `openai_compatible.go`: Generic OpenAI API model implementation

**Interactive Session**: `internal/interactive/session.go`
- Maintains conversation history with system/user/assistant messages
- Two modes:
  1. Interactive: Continuous dialogue with command execution
  2. Pipe: Analyze piped data and exit (no command execution)
- Recursion depth limit (10) prevents infinite command loops
- Truncates large outputs to fit LLM context windows (400k chars limit for both modes)

**Command Executor**: `internal/executor/executor.go`
- Extracts commands from AI responses using `[cmd:query]` and `[cmd:modify]` markers
- Classifies commands by risk level:
  - Query commands (green): Read-only operations, single confirmation (selection list)
  - Modify commands (red): Write operations, double confirmation (selection list)
- Executes commands via `sh -c` shell

**Prompt System**: `internal/prompt/`
- Three prompt types for different scenarios:
  1. Interactive: Initial user question
  2. ContinueAnalysis: Analyzing command output from previous step
  3. PipeAnalysis: Analyzing piped command output
- Language-specific prompts (Chinese/English)

**System Info**: `internal/sysinfo/`
- Collects OS, kernel, available tools information
- Cached in `~/.aiassist/sysinfo.json` for performance
- Added as context to LLM prompts for better command suggestions

**Internationalization**: `internal/i18n/`
- Message translations in `messages_zh.go` and `messages_en.go`
- All user-facing messages go through i18n translator

**UI Utilities**: `internal/ui/`
- Spinner animations
- Terminal separators

## Key Implementation Details

### Model Fallback Order

The order of model calls is **determined by the order in the config file**. For example:

```yaml
providers:
  - name: bailian  # First tried
    models:
      - name: qwen-plus  # 1st choice
        enabled: true
      - name: qwen-max   # 2nd choice
        enabled: true
  - name: openai   # Second tried if bailian fails
    models:
      - name: gpt-4      # 3rd choice
        enabled: true
```

The manager tries each enabled model in this exact order. If a model returns an error, it's marked unavailable and skipped.

### Command Execution Flow

1. User asks question
2. AI analyzes with system prompt + conversation history
3. AI responds with commands marked as `[cmd:query]` or `[cmd:modify]`
4. Executor extracts commands, displays with color coding
5. User confirms (once for query, twice for modify)
6. Command executes, output added to conversation history
7. AI analyzes output and suggests next steps (recursively, max depth 10)

### Pipe Mode vs Interactive Mode

**Pipe mode** (`cmd | aiassist`):
- Reads up to 1.6MB of data
- Analyzes with specialized prompt
- Shows analysis and exits (no command execution)
- No interactive loop

**Interactive mode** (`aiassist`):
- Continuous dialogue
- Executes commands with user confirmation
- Maintains full conversation history
- Max 10 levels of command recursion


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [llaoj/aiassist](https://github.com/llaoj/aiassist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
