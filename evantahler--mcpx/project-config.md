---
trigger: always_on
description: Discover and use MCP tools via the mcpx CLI
---


# mcpx — MCP Tool Discovery and Execution

You have access to external tools via `mcpx`. Use this workflow:

## 1. Search for tools

```bash
mcpx search "<what you want to do>"
```

## 2. Inspect the tool schema

```bash
mcpx info <server> <tool>
```

This shows parameters, types, required fields, and the full JSON Schema.

## 3. Execute the tool

```bash
mcpx exec <tool> '<json args>'                # server auto-resolved if unambiguous
mcpx exec <server> <tool> '<json args>'       # explicit server (required if tool name exists on multiple servers)
mcpx exec <server> <tool> -- --field value    # shell-flag args (typed via the tool's input schema)
mcpx exec <server> <tool> -f params.json
```

Output is JSON by default. Use `--json` to force JSON output in any context — prefer this when you need to parse results programmatically. Use `--format markdown` for rich terminal rendering with colors, headings, and bullet lists.

## Rules

- Always search before executing — don't assume tool names exist
- Always inspect the schema before executing — validate you have the right arguments
- Use `mcpx search -k` for exact name matching
- Pipe results through `jq` when you need to extract specific fields
- Use `--json` when parsing output programmatically (automatic when piped, but explicit is safer)
- Use `--format markdown` for rich terminal-rendered output with colors and formatting
- Use `-v` for verbose debugging (HTTP details + JSON-RPC protocol messages) if an exec fails unexpectedly
- Use `-l debug` to see all server log messages, or `-l error` for errors only

## Examples

```bash
# Find tools related to sending messages
mcpx search "send a message"

# See what parameters Slack_SendMessage needs
mcpx info arcade Slack_SendMessage

# Send a message (server optional if tool name is unique)
mcpx exec Slack_SendMessage '{"channel":"#general","message":"hello"}'

# Or explicitly specify the server
mcpx exec arcade Slack_SendMessage '{"channel":"#general","message":"hello"}'

# Shell-flag form (anything after `--` is parsed against the tool's input schema)
mcpx exec arcade Slack_SendMessage -- --channel "#general" --message "hello"

# Chain commands — search repos and read the first result
mcpx exec github search_repositories '{"query":"mcp"}' \
  | jq -r '.content[0].text | fromjson | .items[0].full_name' \
  | xargs -I {} mcpx exec github get_file_contents '{"owner":"{}","path":"README.md"}'

# Read args from stdin
echo '{"path":"./README.md"}' | mcpx exec filesystem read_file

# Read args from a file with --file flag
mcpx exec filesystem read_file -f params.json
```

## Troubleshooting

- **"Not authenticated" / 401 error** → Run `mcpx auth <server>` to start the OAuth flow
- **Exec timeout** → Use `-v` to see where it stalls; set `MCP_TIMEOUT=<seconds>` to increase the timeout (default: 1800)
- **Search returns no results** → Try `mcpx search -k "*keyword*"` for glob matching, or `mcpx index` to rebuild the search index
- **Missing or stale tools** → Run `mcpx index` to rebuild; any command that connects to a server also auto-updates the index
- **Server won't connect** → Run `mcpx ping <server>` to check connectivity; use `-v` for protocol-level details
- **Auth token expired** → Run `mcpx auth <server> -r` to force a token refresh

## Long-running tools (Tasks)

Some tools support async execution via MCP Tasks. mcpx auto-detects this.

```bash
# Default: waits for the task to complete, showing progress
mcpx exec my-server long_running_tool '{"input": "data"}'

# Return immediately with a task handle (for scripting/polling)
mcpx exec my-server long_running_tool '{"input": "data"}' --no-wait

# Check task status / retrieve result / cancel
mcpx task get my-server <taskId>
mcpx task result my-server <taskId>
mcpx task cancel my-server <taskId>
mcpx task list my-server
```

## Elicitation (Server-Requested Input)

Some servers request user input mid-operation. mcpx handles this automatically in interactive mode. Use `-N` / `--no-interactive` to decline all elicitation (for scripts/CI), or `--json` to handle elicitation programmatically via stdin/stdout.

## 6. Self-authorize (if needed)

Cursor prompts you for every `mcpx exec` call. You can grant yourself granular permissions:

```bash
mcpx allow <server> --cursor              # all tools on a server
mcpx allow <server> <tool> --cursor       # specific tool
mcpx allow --all-read --cursor            # search, info, list, etc.
mcpx allow --all --cursor                 # all mcpx exec calls
mcpx allow --list --cursor                # show current permissions
mcpx deny <server> --cursor               # revoke server permissions
mcpx deny --all --cursor                  # revoke all permissions
```

This writes `Shell(mcpx exec:server:*)` patterns to `.cursor/cli.json`.

## Authentication

```bash
mcpx auth <server>             # authenticate via browser
mcpx auth <server> -s          # check token status and TTL
mcpx auth <server> -r          # force token refresh
mcpx auth <server> --no-index  # authenticate without rebuilding search index
mcpx deauth <server>           # remove stored auth
```

## Available commands

| Command                                | Purpose                           |
| -------------------------------------- | --------------------------------- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [evantahler/mcpx](https://github.com/evantahler/mcpx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
