---
trigger: always_on
description: Generates short session title from user messages. Hooks `before_agent_start`, accumulates recent user prompts, calls `complete()` with active model in background (non-blocking), sets name via `pi.setSessionName()`. Title locks once total accumulated user prompt content (current + previous messages) reaches 40 chars. Max title length: 40 chars.
---

# AGENTS.md

Guidelines for AI coding agents working in this repository.

## What this repo is

A pnpm monorepo of [pi coding agent](https://github.com/earendil-works/pi) extensions. Each package under `packages/` is a self-contained pi extension that can be installed by users into their pi config.

## Packages

### `pi-fence` (`packages/pi-fence`)
Hooks into pi's `tool_call` / `tool_result` events to detect decorative fence/divider comments being written (e.g. `// ---- section ----`, `// ===== Title =====`). Operates in three modes controlled by the `pi-fence-mode` flag:

- **warn** (default) - appends a warning to the tool result after writing
- **block** - returns `{ block: true }` before the write happens
- **remove** - strips the fence comments from the content before writing

Uses [tree-sitter](https://tree-sitter.github.io/) to parse comment nodes. Supports JS/TS, Python, Go, and Rust.

### `pi-caveman` (`packages/pi-caveman`)
Makes the agent respond in caveman mode - cuts ~75% of output tokens while keeping full technical accuracy. Injects a level-specific instruction file into the system prompt at session start. Level is controlled by the `pi-caveman` flag (`lite`, `full`, `ultra`, or `off`; default: `full`).

### `pi-yagni` (`packages/pi-yagni`)
Injects YAGNI ("You Aren't Gonna Need It") discipline into the system prompt at session start. Gives the agent a decision ladder (build only if needed, reuse before writing, stdlib/platform/existing-dep before new code, one line if possible) plus rules against speculative generality and premature abstraction. No configuration — enabled whenever installed. Structured like `pi-caveman` (single instruction file injected via `before_agent_start`) but with no level flag.

### `pi-plan` (`packages/pi-plan`)
Adds a `review-plan` tool that writes a named markdown plan to `~/.pi/plan/<repo>/<name>.md`, commits it to a git repo inside `~/.pi/plan/`, and shows an interactive terminal widget so the user can confirm, request changes, or reply freely before the agent proceeds. The widget also offers an **Open in [Editor]** option (Zed, VS Code, Cursor, Windsurf) detected automatically via `TERM_PROGRAM`; selecting it opens the file in the IDE and re-shows the widget without notifying the agent.

### `pi-cwd` (`packages/pi-cwd`)
Detects absolute cwd paths in `read`, `write`, `edit`, and `bash` tool calls. Appends tip to tool result reminding agent to use relative paths and showing current `cwd`. Also injects `PROMPT_INSTRUCTIONS` into system prompt at session start.

Controlled by `pi-cwd` boolean flag (default: `true`).

### `pi-subagents` (`packages/pi-subagents`)
Lets the agent spawn specialized subagents — each in its own isolated session with its own model, tools, skills, and instructions. Templates are markdown files with YAML frontmatter stored in `~/.pi/agents/subagents/<name>.md` (global) or `.pi/subagents/<name>.md` (project). A project template overrides a global one with the same name.

Exposes a single `subagent` tool:
- **spawn**: call with `name`, `description`, `prompt` — blocks until the subagent finishes, returns its result
- **follow-up**: call again with the same `id` from a previous result plus a new `prompt` to continue the session

Subagents cannot spawn further subagents. Concurrency is capped at `pi-subagents-max-concurrent` (default: 4); excess agents queue and start as slots free.

Frontmatter fields: `description`, `model`, `thinking`, `max_turns`, `grace_turns`, `included_tools`, `included_skills`, `included_subagents`, `enabled`.

List fields (`included_tools`, `included_skills`, `included_subagents`) accept YAML array syntax (preferred) or CSV string (backward compat):

```yaml
# preferred — idiomatic YAML
included_tools:
  - edit
  - write
  - bash

# also accepted — flow array
included_tools: [edit, write, bash]

# also accepted — CSV string (legacy)
included_tools: edit, write, bash
```

The `subagent:templates` command opens an interactive terminal menu listing all loaded templates with their source and model.

### `pi-title` (`packages/pi-title`)
Generates short session title from user messages. Hooks `before_agent_start`, accumulates recent user prompts, calls `complete()` with active model in background (non-blocking), sets name via `pi.setSessionName()`. Title locks once total accumulated user prompt content (current + previous messages) reaches 40 chars. Max title length: 40 chars.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [piotr-oles/pi-extensions](https://github.com/piotr-oles/pi-extensions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
