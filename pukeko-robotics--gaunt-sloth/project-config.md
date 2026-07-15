---
trigger: always_on
description: This file provides guidance to any AI coding agent (Claude Code, Cursor, etc.) working with this repository.
---

# Gaunt Sloth Internal Development Guidelines

This file provides guidance to any AI coding agent (Claude Code, Cursor, etc.) working with this repository.

## Technologies Used

- NodeJS 22 (LTS)
- Vitest 3 for tests
- Typescript 5
- LangChain and LangGraph 0.3

Please refer to package.json to check exact versions

## Core Development Principles

Vendor and system abstractions and wrappers should be used in most cases.

### UX / TUI guidelines

Any change to the terminal UI (`packages/app/src/tui/`) or user-facing CLI feedback must follow
the **[TUI / CLI UX Guidelines](docs/ux-guidelines.md)** — the code-grounded ruleset for command
notices, `/clear` behaviour, tool-call panels, markdown, layout, the keyboard model, and colour
semantics. It implements Project TAKAHĒ's cross-surface Design Language (the `DL-n` principles).
When you add or change a user-facing behaviour, cite the DL principle it serves and update that doc.

### Imports

Project uses import alias with `#src/*.js` pointing to `src/` and after build resolving to generated `dist/`.
Please abstain from using relative imports, only use them when no other choices are available
(currently the only exception is entry point cli.js)

### Architecture and Flow

- Make sure proper separation of LangChain components (LLMs, chains, agents, tools)
- Check for clear data flow between components
- Ensure proper state management in LangGraph workflows
- Validate error handling and fallback mechanisms

### Security

- Make sure API key handling and environment variables
- Make sure no personal data is present in code
- **Make sure that API keys are NOT accidentally included into diff.**
- Check for proper input sanitization
- Verify output validation and sanitization

### Output

Use [consoleUtils.ts](src/consoleUtils.ts) to output to users.
Do not use console.log directly.

### System

Use [systemUtils.ts](src/systemUtils.ts) to access system variables and functions such as
process.env, process.stdout, etc.

### LLM

Use [llmUtils.ts](src/llmUtils.ts) to access LLM.

### Middleware

Starting with v1.0.0, Gaunt Sloth uses LangChain middleware pattern instead of hooks.

Middleware provides hooks to intercept and control agent execution at critical points:
- `beforeModel`: Called before model invocation
- `afterModel`: Called after model response
- `beforeAgent`: Called before agent initialization
- `afterAgent`: Called after agent completion
- `wrapModelCall`: Wrap model calls with full control
- `wrapToolCall`: Wrap tool calls with full control

**Predefined Middleware:**
- `anthropic-prompt-caching`: Reduces API costs by caching prompts (Anthropic only)
- `summarization`: Condenses conversation history when approaching token limits

**Configuration:**
- Middleware is configured in the `middleware` array in config
- JSON configs support predefined middleware (string or config object)
- JS configs support both predefined and custom middleware objects

**Implementation:**
- Middleware registry is in [src/middleware/registry.ts](src/middleware/registry.ts)
- Middleware types are in [src/middleware/types.ts](src/middleware/types.ts)
- Provider-specific middleware can be auto-injected via `postProcessJsonConfig()` in preset files

### AG-UI Server (`@gaunt-sloth/api`)

`startAgUiServer()` ([packages/api/src/modules/apiAgUiModule.ts](packages/api/src/modules/apiAgUiModule.ts))
exposes the agent over the AG-UI protocol at `POST /agents/:agentId/run`,
streaming typed SSE events. It is intended for **local clients only** (a local
web UI talking to a local CLI agent); do not expose it to public networks.

Request handling:
- A request carrying `forwardedProps.command.resume` resumes a graph suspended
  by `interrupt()` (client-fulfilled tools); otherwise it starts a fresh run.
- The client is the source of truth for history: it sends the full message list
  every turn, and LangGraph's `add_messages` reducer dedupes by message id, so
  re-sending prior messages does not duplicate state on the checkpointer.

**Be defensive when converting client messages.** A single malformed message
must never abort a run — because it is part of the persisted history, it would
otherwise poison every subsequent turn on the thread. In particular, tool-call
`arguments` strings are parsed via `parseToolArguments()`, not raw `JSON.parse`:
local models (Ollama/Gemma) do not honor `disable_parallel_tool_use`, and their
streamed delta reassembly can concatenate sibling calls' argument buffers into
invalid JSON such as `{}{}` or `{"steps":3}{}`. The parser recovers the first
complete JSON value, warns via `displayWarning`, and falls back to `{}`. Keep
new client-message parsing on this resilient path.

## Tool Use

Precedence for your tool use:
1. Your built-in tools (e.g. Read, Edit, Write, Glob, Grep, etc.)
2. Bash commands that are documented in this file and in README.md
3. Other bash commands

**Examples of what to avoid:**
- ❌ `cat file.txt` → ✅ Use Read tool
- ❌ `grep pattern file.txt` → ✅ Use Grep tool
- ❌ `echo content > file.txt` → ✅ Use Write tool
- ❌ `find . -name "*.js"` → ✅ Use Glob tool

Abstain from using bash commands when you already have a built-in tool,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pukeko-robotics/gaunt-sloth](https://github.com/pukeko-robotics/gaunt-sloth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
