---
trigger: always_on
description: Boardstate is agent-native in **two directions**:
---

# Agents & Boardstate

Boardstate is agent-native in **two directions**:

| Direction                       | What it means                                                                                                              | Packages                                   |
| ------------------------------- | -------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| **Agents build the board**      | Any AI composes tabs, widgets, bindings, and custom sandboxed widgets through the `boardstate_*` tool set                  | `@boardstate/mcp`, `@boardstate/agent`     |
| **The board acts on the world** | The board consumes _external_ MCP servers: their reads become live data, their mutations become operator-confirmed actions | `@boardstate/broker`, `@boardstate/server` |

This file covers how to set each surface up, the full tool catalog, the conventions that
make agent-built boards good, and the rules for coding agents working on this repo.

---

## 1. Give any AI the board (`@boardstate/mcp`)

The MCP server exposes the complete dashboard tool set to any MCP client — Claude Desktop,
Claude Code, or anything else that speaks MCP. State persists to `$BOARDSTATE_STATE_DIR`
(default `~/.boardstate`), so the same board is shared by the CLI, the MCP server, and any
host you run.

**Claude Code** (one command):

```sh
claude mcp add boardstate -- npx -y @boardstate/mcp
```

**Claude Desktop** (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "boardstate": { "command": "npx", "args": ["-y", "@boardstate/mcp"] }
  }
}
```

**Watch the board the agent is building** — add `--serve` and open the printed page; it
live-updates as tools land:

```sh
npx @boardstate/mcp --serve 4400
```

Inside MCP-Apps-capable clients, `boardstate_board_view` renders the live board directly
in the conversation.

### The tool catalog

| Group           | Tools                                                                                                                             |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| Read & discover | `boardstate_workspace_get` · `boardstate_widget_catalog` · `boardstate_data_read` · `boardstate_board_view`                       |
| Tabs            | `boardstate_tab_create` · `boardstate_tab_update` · `boardstate_tab_delete` · `boardstate_tabs_reorder` · `boardstate_layout_set` |
| Widgets         | `boardstate_widget_add` · `boardstate_widget_update` · `boardstate_widget_move` · `boardstate_widget_remove`                      |
| Custom widgets  | `boardstate_widget_scaffold` (sandboxed; lands as a **pending** card) · `boardstate_widget_approve` (operator surfaces only)      |
| Whole-document  | `boardstate_workspace_replace` (full-document write, sanitized + capability-reconciled) · `boardstate_undo`                       |
| Quality         | `boardstate_design_review` (screenshot + critique the board you just built)                                                       |
| External tools  | `boardstate_tool_search` (SEARCH the connector catalog / REQUEST grants — see §3)                                                 |
| Errors          | `boardstate_error` (structured failure reporting)                                                                                 |

### Conventions for board-building agents

- **Catalog first.** Call `boardstate_widget_catalog` before composing — every builtin has
  a schema-valid example there. Guessing props is the #1 source of rejected calls.
- **[Living answers](docs/living-answers.md).** Answer visual questions with live widgets,
  not prose. A question about revenue deserves a bound chart, not a paragraph.
- **[Composition patterns](docs/composition-patterns.md).** Which builtin for which job,
  when to scaffold a custom widget, layout rules of thumb.
- **[Design review](docs/design-review.md).** After building, run `boardstate_design_review`
  and fix what it finds — the self-building loop.
- **Provenance is tracked.** Every tab/widget carries a `createdBy` actor; write honest ones.

**Ready-made skill:** all of the above is packaged as
[`.claude/skills/using-boardstate/SKILL.md`](.claude/skills/using-boardstate/SKILL.md) —
Claude Code picks it up automatically in this repo; copy it into your own project's
`.claude/skills/` (or `~/.claude/skills/`) to teach any Claude session, or paste it into
the system prompt of a bare-API harness (GLM, any OpenAI-compatible loop). If you embed
`@boardstate/agent`, you don't need it — its system prompt already carries these
conventions.

## 2. Embed a chat agent in your host (`@boardstate/agent`)

`@boardstate/agent` is the embeddable agent loop the [live app](https://100yenadmin.github.io/boardstate/app/)
uses: streaming chat, the full tool loop, the composition system prompt, and optional
self-review — bring your own provider key.

```js
import { createAgentChatAgent, anthropicAdapter, openAICompatAdapter } from "@boardstate/agent";

// Anthropic
const provider = anthropicAdapter({
  apiKey: process.env.ANTHROPIC_API_KEY,
  model: "claude-sonnet-5",
});

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [100yenadmin/boardstate](https://github.com/100yenadmin/boardstate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
