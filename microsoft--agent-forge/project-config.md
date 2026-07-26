---
trigger: always_on
description: This file guides the **GitHub Copilot CLI** (the generation engine) to produce artifacts in the correct format. All generated files are **VS Code-compatible** and also work in GitHub Copilot CLI at runtime.
---


# AGENT-FORGE — Output Format Specification

This file guides the **GitHub Copilot CLI** (the generation engine) to produce artifacts in the correct format. All generated files are **VS Code-compatible** and also work in GitHub Copilot CLI at runtime.

## Agent File Format (`.agent.md`)

```yaml
---
name: "Display Name"                    # REQUIRED
description: "What this agent does"     # REQUIRED
argument-hint: "[task] [details]"       # Recommended — placeholder in chat input
tools:                                  # Array of tool aliases (see Tool Aliases below)
  - read                                #   Read file contents (aliases: Read, NotebookRead)
  - edit                                #   Modify files (aliases: Edit, MultiEdit, Write, NotebookEdit)
  - search                              #   Grep/glob search (aliases: Grep, Glob)
  - execute                             #   Shell/terminal commands (aliases: shell, Bash, powershell, run_in_terminal)
  - agent                               #   Invoke sub-agents (aliases: custom-agent, Task)
  - web                                 #   Fetch URLs (aliases: WebSearch, WebFetch)
  - todo                                #   Task lists (aliases: TodoWrite)
agents:                                 # List of allowed subagent names (orchestrators only)
  - "researcher"                        #   Only these agents can be invoked as subagents
  - "implementer"                       #   Use '*' to allow all, '[]' to prevent any
model: "Claude Sonnet 4.5 (copilot)"   # Optional: single model or prioritized array
# model:                                # Array form — tries each in order until available:
#   - "Claude Sonnet 4.5 (copilot)"
#   - "Gemini 3 Flash (Preview) (copilot)"
user-invokable: true                    # Show in agent dropdown (default: true)
disable-model-invocation: false         # Prevent auto-delegation (default: false)
target: "vscode"                        # Optional: "vscode" or "github-copilot" (both if omitted)
handoffs:                               # Multi-agent workflow transitions (flat pattern)
  - label: "Hand off to Backend"
    agent: "express"
    prompt: "Continue with backend work"
    send: false
---

Body: markdown instructions for the agent.
```

### Subagent Properties

| Property | Purpose | Default |
|----------|---------|---------|
| `agents` | List of allowed subagent names for this agent. Use `'*'` to allow all, `[]` to prevent any. Only meaningful when `agent` tool is included. | `'*'` (all) |
| `model` | AI model for this agent. String or prioritized array. Useful for cost-efficient subagents. | Inherits from session |
| `user-invokable` | Whether agent appears in the agents dropdown. Set `false` for subagent-only agents. | `true` |
| `disable-model-invocation` | Whether to prevent other agents from auto-invoking this as a subagent. Set `true` for orchestrators (user-invoked only). | `false` |

### Tool Aliases Reference

| Alias | Platform Equivalents | Description |
|-------|---------------------|-------------|
| `execute` | shell, Bash, powershell | Execute shell commands |
| `read` | Read, NotebookRead | Read file contents |
| `edit` | Edit, MultiEdit, Write, NotebookEdit | Modify files |
| `search` | Grep, Glob | Search files or text |
| `agent` | custom-agent, Task | Invoke other agents |
| `web` | WebSearch, WebFetch | Fetch URLs, web search |
| `todo` | TodoWrite | Create/manage task lists |
| `github/*` | — | GitHub MCP server tools |
| `playwright/*` | — | Playwright MCP server tools |

Unrecognized tool names are ignored, enabling cross-product compatibility.

## Custom Instructions Discovery

At runtime, GitHub Copilot (in VS Code and Copilot CLI) discovers and loads instructions from these locations:

| Location | Scope |
|----------|-------|
| `~/.copilot/copilot-instructions.md` | Global (all sessions) |
| `.github/copilot-instructions.md` | Repository |
| `.github/instructions/**/*.instructions.md` | Repository (modular) |
| `AGENTS.md` (Git root or cwd) | Repository |
| `Copilot.md`, `GEMINI.md`, `CODEX.md` | Repository |

Repository instructions take precedence over global instructions.

## Instruction File Format (`.instructions.md`)

```yaml
---
name: "instruction-slug"               # REQUIRED
description: "What standards are enforced"  # REQUIRED
applyTo: "**/*.{ts,tsx,js,jsx}"        # REQUIRED — glob for auto-loading
---

Body: rules grouped under ## headings, bullet points with reasoning.
```

**applyTo must be specific** — `**/*` wastes context. Use file-type patterns.

## Skill File Format (`SKILL.md`)

```yaml
---
name: "skill-slug"                      # REQUIRED — must match parent directory name
description: "Domain knowledge. USE FOR: 5+ trigger phrases. DO NOT USE FOR: 3+ exclusions."  # REQUIRED (1-1024 chars)
argument-hint: "[topic or context]"       # Hint shown in chat input for /slash command
user-invokable: true                      # Show in /slash menu (default: true)
disable-model-invocation: false           # Allow auto-loading (default: false)
license: "MIT"                            # Optional license

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/agent-forge](https://github.com/microsoft/agent-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
