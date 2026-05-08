---
trigger: always_on
description: Interactive manager CLI (and VSCode extension) that uses **Codex CLI** as a controller and **Claude Code CLI** as a worker in a supervised agentic loop.
---

# QA Panda

Interactive manager CLI (and VSCode extension) that uses **Codex CLI** as a controller and **Claude Code CLI** as a worker in a supervised agentic loop.

## Architecture

```
User input
  -> Codex CLI (controller) decides what to do
    -> Claude Code CLI (worker) executes the task
      -> Controller reviews result, loops or stops
```

The controller (Codex) receives user messages and outputs structured JSON decisions with `action: "delegate"` or `action: "stop"`, along with `claude_message` (prompt for the worker) and `controller_messages` (messages shown to the user). The worker (Claude Code) executes the prompt using `--output-format stream-json` and streams results back.

### Core loop (`src/orchestrator.js`)

1. User sends a message
2. Controller turn: Codex CLI runs, reads transcript, outputs a JSON decision
3. If decision is `stop`, the loop ends
4. If decision is `delegate`, the worker turn starts: Claude Code runs with the `claude_message`
5. Worker result is appended to transcript, loop returns to step 2

## Project Structure

```
bin/qapanda.js          Entry point (CLI)
src/
  cli.js                   CLI argument parsing, subcommands (shell, run, resume, status, logs, list, doctor)
  shell.js                 Interactive readline shell (terminal UI)
  orchestrator.js          Core manager loop (controller turn -> worker turn -> repeat)
  codex.js                 Spawns Codex CLI, parses structured JSON output
  claude.js                Spawns Claude Code CLI, handles streaming events
  events.js                Parses raw streaming events into normalized summaries
  render.js                Terminal renderer (ANSI colors, timeline UI with circles/pipes)
  state.js                 Run state management (manifests, run dirs, requests, loops)
  schema.js                JSON schema for controller output format
  prompts.js               System prompt for the controller
  process-utils.js         Process spawning helpers (Windows .cmd shim handling)
  utils.js                 General utilities (JSON I/O, truncate, slugify, etc.)
extension/
  package.json             VSCode extension manifest
  extension.js             Extension activate/deactivate, WebviewPanel creation
  webview-renderer.js      WebviewRenderer (same API as terminal Renderer, sends postMessage)
  session-manager.js       Bridges webview input to orchestrator (replaces shell.js)
  webview/
    main.js                Frontend: renders messages, markdown, thinking animation
    style.css              Timeline UI styles, role colors, input area
  resources/
    icon.svg               Orange "CC" icon for editor title bar
  .vscodeignore            Excludes dev files from vsix package
  .gitignore               Ignores copied src/, .vsix, node_modules
```

## Key Concepts

### Renderer interface

Both `src/render.js` (terminal) and `extension/webview-renderer.js` (VSCode) implement the same public API:

- `user(text)`, `controller(text)`, `claude(text)`, `shell(text)` - Role-labeled messages
- `claudeEvent(raw)`, `controllerEvent(raw)` - Process raw streaming events
- `streamMarkdown(label, text)`, `flushStream()` - Streaming text output
- `launchClaude(prompt, sameSession)` - Show worker launch info
- `stop()`, `close()` - Lifecycle
- `banner(text)`, `line(label, text)`, `mdLine(label, text)` - Misc output
- `requestStarted(runId)`, `requestFinished(message)` - Request lifecycle
- `userPrompt()` - Returns prompt string (terminal) or no-op (webview)
- `write(text)` - Raw output (terminal only)

### Claude Code streaming events

Claude Code outputs `stream-json` with these event types we care about:
- `content_block_delta` with `text_delta` -> text streaming
- `content_block_delta` with `input_json_delta` -> tool input accumulation
- `content_block_start` with `tool_use` -> tool call start (name, index)
- `content_block_stop` -> block finished (triggers tool call formatting)

### Tool call formatting

Tool calls are accumulated (start -> input deltas -> stop) then formatted with human-readable descriptions:
- `Bash` -> "Running command: ..."
- `Read` -> "Reading path"
- `Write/Edit` -> "Writing/Editing path"
- `Glob` -> "Glob: pattern"
- `Grep` -> "Grep: pattern in path"

### Terminal timeline UI (`src/render.js`)

- Unicode circles (U+25CF) on content lines, pipes (U+2502) between them
- Role headers: User (cyan), Controller (yellow), Claude code (green), Shell (magenta)
- User messages have gray background, controller messages have dark yellow background
- Inline markdown to ANSI conversion (bold, italic, code, headings, bullets)

### Windows process spawning (`src/process-utils.js`)

- `winEscapeArg()` handles proper quoting for Windows .cmd shims
- `spawnArgs()` wraps args with `shell: true` on Windows
- Both `codex` and `claude` are spawned as child processes with streaming stdout

### State management (`src/state.js`)

Runs are stored in `.qpanda/runs/<run-id>/`:
- `manifest.json` - Run metadata, config, counters, request history
- `events.jsonl` - All events (user messages, controller decisions, worker results)
- `transcript.jsonl` - Conversation transcript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gzmagyari/qapanda](https://github.com/gzmagyari/qapanda) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
