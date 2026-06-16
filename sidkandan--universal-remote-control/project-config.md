---
trigger: always_on
description: <!-- This file contains instructions for Gemini CLI agents. For human documentation, see README.md -->
---

<!-- This file contains instructions for Gemini CLI agents. For human documentation, see README.md -->
# URC — CLI-to-CLI Communication & RC Bridge

## What This Is
URC enables cross-CLI communication between Claude Code, Codex, and Gemini panes via MCP tools backed by a shared SQLite coordination database.

## RC Bridge
The RC Bridge makes this Gemini pane controllable from the Claude Code phone app. A Haiku relay pane forwards messages between the phone app and this pane.

**How it works:**
1. A Claude Code relay pane runs the `rc-bridge` agent (Haiku model)
2. It's pre-configured with target pane ID and CLI type via tmux pane options (set by `urc-spawn.sh`)
3. State is stored in tmux pane options (`@bridge_target`, `@bridge_cli`, `@bridge_relays`)
4. Phone message -> relay hook dispatches to this pane via `send.sh` -> your response is captured by `hook.sh` and written to a push file -> relay picks it up on next wake and displays it verbatim on phone

**Launch:** Use `/rc` command in this Gemini pane, or `/rc-bridge gemini` from Claude Code.

## Available MCP Tools (urc_coordination — 11 tools)

> **Note:** Gemini CLI uses underscores in MCP server names (`urc_coordination`) instead of hyphens (`urc-coordination`) — this is a Gemini CLI naming convention requirement. The naming is transparent: just use `mcp__urc_coordination__tool_name()` syntax. The MCP server is `urc/core/server.py`.

**NEVER run `python3 urc/core/server.py` as a shell command** — it is an MCP server, not a CLI tool.

| Tool | Purpose |
|------|---------|
| `register_agent` | Register this pane in the coordination DB |
| `heartbeat` | Send a heartbeat with context usage |
| `get_fleet_status` | List all registered agents |
| `rename_agent` | Set a display label for a pane |
| `dispatch_to_pane` | Send a message to a tmux pane via send.sh |
| `read_pane_output` | Capture recent output from a pane's buffer |
| `send_message` | Send a message to another agent (use `notify=true` to include wake nudge) |
| `receive_messages` | Get unread messages |
| `kill_pane` | Kill a tmux pane (requires explicit confirmation) |
| `cancel_dispatch` | SIGINT target + clear signals + unblock waiting dispatcher |
| `bootstrap_validate` | Validate CWD/directories/hook/tmux setup |

## Turn Completion
The `AfterAgent` hook in `.gemini/settings.json` fires `hook.sh` after every turn. This performs the full 4-step signal ordering:
1. Write response file `.urc/responses/{PANE}.json`
2. Touch signal file `.urc/signals/done_{PANE}`
3. `tmux wait-for -S "urc_done_{PANE}"` (wakes blocking dispatchers)
4. Append JSONL `.urc/streams/{PANE}.jsonl`

## Verifying MCP Connectivity

To check that MCP servers are properly connected in a running Gemini session:

- **`/mcp list`** — The correct way to verify MCP connectivity. Shows configured servers, connection state, discovered tools/prompts/resources, and per-server errors. This is the canonical MCP health check.
- **`/mcp refresh`** — Restarts MCP servers and reloads tools without restarting Gemini. Use this after config changes.
- **`/mcp desc`** or **`/mcp schema`** — Deeper validation of tool descriptions and schemas.
- **Do NOT use `/tools`** to check MCP status. The `/tools` command intentionally filters out MCP tools (`serverName` tools are excluded), so it will show zero MCP tools even when servers are connected and working.
- **Out-of-session check:** `gemini mcp list` runs an active connection test (connect + ping with 5s timeout) and reports Connected/Disconnected per server.

## Pane Dispatch Rules
- **Never use raw `tmux send-keys`** — always use `dispatch_to_pane` MCP tool or `send.sh`
- **NEVER run `python3 urc/core/server.py` as a shell command** — it is an MCP server, not a CLI tool. Running it starts an infinite STDIO loop that hangs your shell. Use MCP tools or `tmux capture-pane -t %NNN -p -S -80` instead.
- **Keep messages short** — under ~1000 chars for dispatched text (see Message size limits in Cross-Pane Communication). Use handoff files for longer content.
- **Check return status** — `delivered` = success, `failed` = pane dead
- **Verify after dispatch** — wait 5-10s, then `read_pane_output()` to confirm processing started
- **Cold-start delay** — freshly spawned Gemini panes need ~10s to initialize. The relay flow handles this automatically, but direct dispatch to a new Gemini pane may fail if sent within 10s of pane creation.
- Core edits (`urc/core/`) require planning first

## Inbox & Bidirectional Messaging

Gemini has automatic inbox notification via the `BeforeAgent` hook (`hooks/scripts/inbox-check.sh gemini`). This fires before every turn — no polling needed. When messages are waiting, you'll see:
```
INBOX: You have 2 unread message(s) from %1316. Call receive_messages("%YOUR_PANE") to read them.
```

**INBOX notifications are high priority — act immediately:**
1. Call `mcp__urc_coordination__receive_messages(pane_id="%YOUR_PANE")` before doing anything else
2. Read the messages and act on them before continuing other work
3. Reply using DB messaging (see [Communication Strategy](#communication-strategy) for when to use each approach)

**Sending messages to other agents:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sidkandan/universal-remote-control](https://github.com/sidkandan/universal-remote-control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
