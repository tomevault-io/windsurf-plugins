---
trigger: always_on
description: Agentic coding CLI that connects to Anthropic API (or compatible endpoints) to provide a terminal-based Claude Code experience. 47 source files, ~9.7k lines, single runtime dependency (`fast-glob`).
---

# Clio — Claude Code CLI Clone

Agentic coding CLI that connects to Anthropic API (or compatible endpoints) to provide a terminal-based Claude Code experience. 47 source files, ~9.7k lines, single runtime dependency (`fast-glob`).

## Quick Reference

```bash
npm run dev              # Run without build (tsx)
npm run build            # Compile TypeScript
npm run test             # Run all tests (vitest)
npm run test -- --watch  # Watch mode
npm run test -- tests/permissions.test.ts  # Single file
```

## Architecture

```
src/
├── index.ts              # Entry: REPL loop, command routing, print mode (-p)
├── types.ts              # Shared types (Config, Message, ContentBlock, UsageStats)
├── core/                 # Engine (18 files)
│   ├── agent.ts          # Agent loop (25 max iterations, auto-compact at 85%)
│   ├── client.ts         # HTTP client (SSE streaming, retry 429/502/503/504)
│   ├── system-prompt.ts  # 14-section system prompt builder
│   ├── section-cache.ts  # Static sections cached per session, dynamic recomputed
│   ├── context.ts        # CLAUDE.md upward traversal + git context
│   ├── settings.ts       # 4-layer config merge (global → project × committed/local)
│   ├── permissions.ts    # 3 modes (default/auto/plan) + allow/deny rules
│   ├── llm-classifier.ts # 2-stage: pattern match → LLM fallback
│   ├── session.ts        # Session persistence (~/.clio/sessions/)
│   ├── compact.ts        # Context compaction via summarization
│   ├── normalize.ts      # Message normalization (tool pairing, thinking blocks)
│   ├── memory.ts         # Auto memory (4 types, MEMORY.md index)
│   ├── adaptive-thinking.ts # Dynamic thinking budget
│   ├── pricing.ts        # Model pricing tables
│   ├── sandbox.ts        # Path/network/env sandboxing
│   ├── billing.ts        # Billing header for CC compatibility
│   └── prompts.ts        # Prompt templates
├── tools/                # Tool system (9 files)
│   ├── index.ts          # 21 tool definitions + executeTool() dispatcher
│   ├── mcp.ts            # MCP server lifecycle (JSON-RPC 2.0, zero deps)
│   ├── lsp.ts            # LSP client (Content-Length framing, zero deps)
│   ├── subagent.ts       # Sub-agent spawning (foreground/background/worktree)
│   ├── teams.ts          # Agent team coordination + SendMessage
│   ├── hooks.ts          # Pre/post tool hooks (shell commands)
│   ├── tasks.ts          # In-memory task store
│   ├── checkpoint.ts     # File undo/rollback
│   └── worktree.ts       # Git worktree isolation
├── ui/                   # Terminal UI (8 files)
│   ├── render.ts         # ANSI colors, spinners, box drawing
│   ├── input.ts          # Raw stdin, CSI parsing, undo/redo, history
│   ├── markdown.ts       # Streaming markdown renderer
│   ├── highlight.ts      # Syntax highlighting (TS/JS/Python/Rust/Go/Bash/JSON/CSS/HTML)
│   ├── statusbar.ts      # Footer bar (model, tokens, cost)
│   ├── keybindings.ts    # Keyboard shortcuts
│   ├── image.ts          # Image detection + base64 encoding
│   └── file-completions.ts # @file tab completion
├── commands/             # Slash commands (4 files)
│   ├── git-commands.ts   # /commit /pr /review
│   ├── init.ts           # /init (generate CLAUDE.md)
│   ├── doctor.ts         # /doctor (diagnostics)
│   └── custom-agents.ts  # Agent definition loading (.md files)
├── plugins/              # Plugin system (4 files)
│   ├── types.ts, manifest.ts, loader.ts, index.ts
└── skills/               # Skills system (3 files)
    ├── index.ts, loader.ts, builtins/index.ts
```

## Key Patterns

### Config hierarchy (settings.ts)
`~/.clio/settings.json` → `~/.clio/settings.local.json` → `.clio/settings.json` → `.clio/settings.local.json`
- Arrays (allowRules, denyRules, hooks) concatenate
- Objects (mcpServers, lspServers) shallow merge
- Scalars override

### Tool registration (tools/index.ts)
Tools are objects in `TOOL_DEFINITIONS[]` with `{ name, description, input_schema }`. Execution via `executeTool(name, input)`. Add new tools by appending to the array and adding a case in the dispatcher.

### System prompt (system-prompt.ts + section-cache.ts)
10 static sections (cached per session) + 4 dynamic sections (recomputed per turn). Reference text: `docs/cc-system-prompt.md`.

### Imports
- Node APIs: `import * as fs from "node:fs/promises"`
- Local: `import { foo } from "./bar.js"` (**.js extension required** for ESM)
- Types: `import type { X } from "../types.js"`

### Error handling
- I/O: try-catch, return null on failure
- Tools: return `{ is_error: true, content: "..." }`
- Network: exponential backoff for 429/502/503/504

## Code Rules

- **TypeScript strict mode** — no `any` (including `as any`, `: any`, `<any>`)
- **Single runtime dependency** — only `fast-glob`. Do not add new deps without discussion
- **ESM** — `"type": "module"` in package.json, `.js` extensions in imports
- **No unnecessary additions** — no docstrings, comments, or type annotations on unchanged code
- After renaming/deleting a symbol, grep `src/` and `tests/` to confirm zero residual references

## Self-iteration Workflow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [icebear0828/clio](https://github.com/icebear0828/clio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
