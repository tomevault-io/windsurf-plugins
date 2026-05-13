---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Crybot - Crystal AI Assistant

Crybot is a modular personal AI assistant built in Crystal, inspired by nanobot (Python). It provides multiple interaction modes (REPL, web, Telegram, voice) and supports multiple LLM providers.

## Build & Development Commands

### Building

```bash
# Build both binaries (recommended)
shards build -Dpreview_mt -Dexecution_context --release

# Or use the Makefile
make build                    # Build crybot with proper flags
make shell-build              # Build crysh (shell wrapper)
make all                      # Build both binaries
make run                      # Build and run crybot
make clean                    # Remove binaries
make deploy_site              # Deploy documentation site

# Manual builds
crystal build src/main.cr -o bin/crybot -Dpreview_mt -Dexecution_context
crystal build src/crysh.cr -o bin/crysh
```

**Note**: `shards build` works with the proper flags. Use `-Dpreview_mt -Dexecution_context` for crybot's multi-threading support. Crysh doesn't require these flags.

### Linting
```bash
ameba --fix           # Auto-fix linting issues
```

### Running
```bash
# Crybot (main assistant)
./bin/crybot              # Start all enabled features (default: threaded mode)
./bin/crybot onboard      # Initialize configuration
./bin/crybot agent [-m <msg>]  # Direct agent interaction
./bin/crybot status       # Show configuration status
./bin/crybot profile     # Profile startup performance
./bin/crybot tool-runner <tool_name> <json_args>  # Internal: execute tool in Landlocked subprocess

# Crysh (shell wrapper)
./bin/crysh "description"  # Generate and execute shell command from natural language
./bin/crysh -y "description"  # Skip rofi confirmation (for scripts)
```

**Note**: There are two binaries - `bin/crybot` (main assistant) and `bin/crysh` (shell wrapper). The `ameba` file in `bin/` is just the development linter.

**Crysh** - Natural language shell wrapper:
- Generates shell commands from natural language descriptions using LLMs
- Shows rofi dialog for confirmation before execution (Run/Edit/Cancel)
- Supports editing via $EDITOR or rofi prompt
- Use `-y` flag to skip confirmation (useful in scripts)
- Preserves stdin/stdout for clean pipeline integration
- Examples:
  - `echo "a,b,c" | crysh "get second field"` → `cut -d, -f2`
  - `ls -l | crysh "sort by size"` → sorts by file size
  - `crysh "count unique lines" < file.txt` → `sort | uniq -c`

### Testing
No test suite exists yet in this repository.

## Architecture Overview

### Entry Point & Commands
- `src/main.cr` - Entry point with **compile-time flag checks** for preview_mt and execution_context
- Commands via docopt: `onboard`, `agent`, `status`, `profile`, `tool-runner` (internal)
- `src/commands/` - Individual command handlers (`threaded_start.cr` is default when no command given)

### Core Systems

**Feature Coordinator** (`src/features/coordinator.cr`)
- Orchestrates multiple independent features running in separate fibers
- Each feature implements `FeatureModule` interface (`start`, `stop` methods)
- Auto-restart on config changes via Process.exec
- Features: gateway (Telegram), web, voice, repl, scheduled_tasks

**Agent Loop** (`src/agent/`)
- `loop.cr` - Main agent loop with tool calling iteration (max iterations configurable)
- `context.cr` - Message history and system prompt construction
- `memory.cr` - Long-term memory (MEMORY.md) and daily logs in workspace/memory/
- `tool_monitor.cr` - **Executes tools in isolated fibers with Landlock sandboxing**
- `tools/` - Built-in tools (filesystem, shell, web, memory, skill_builder)
- `skills/` - Bootstrap skills system with credential support

**Tool Execution & Landlock Sandboxing** (CRITICAL)
- Tools execute in **isolated fibers** using `Fiber::ExecutionContext::Isolated`
- Built-in tools run under **Landlock restrictions** (Linux kernel 5.13+ sandboxing)
- **Landlock denied exceptions** trigger access request via rofi/terminal prompt
- Built-in tools raise `Tools::LandlockDeniedException` for path access requests
- MCP tools (`tool_name.includes?("/")`) execute **directly** (not in Landlocked context)
- **MCP servers require subprocess spawning**, so agent itself cannot run under Landlock
- Access requests go through `LandlockSocket` to monitor process
- Default path rules: read-only (/usr, /bin, /lib, /etc), read-write (~/.crybot/playground, ~/.crybot/workspace, /tmp)

**Providers** (`src/providers/`)
- Abstract `Base` provider interface (`chat` method with streaming support)
- Implementations: OpenAI, Anthropic, Zhipu (GLM), OpenRouter, Groq, Gemini, DeepSeek, vLLM
- Lite mode support for Groq/OpenRouter (disables tool calling)
- Selected via `config.yml` under `agents.defaults.provider` and `.model`

**Configuration** (`src/config/`)
- YAML-based config stored in `~/.crybot/config.yml`
- `loader.cr` - Loads, validates, and runs migrations
- `watcher.cr` - File watcher triggers Process.exec for hot-reload

**Session Management** (`src/session/`)
- `manager.cr` - Singleton pattern for persistent conversation history
- JSONL-based storage in `~/.crybot/sessions/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ralsina/crybot](https://github.com/ralsina/crybot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
