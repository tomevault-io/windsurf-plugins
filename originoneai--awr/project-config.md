---
trigger: always_on
description: AWR does not run your agent. The agent client (Codex, Claude Code, Kimi, Cursor, Grok) does the work
---

# Connect your agent

AWR does not run your agent. The agent client (Codex, Claude Code, Kimi, Cursor, Grok) does the work
with its own tools; AWR supplies current project facts, work context, claims, and recoverable
session memory for the same initialized project. Provider and model labels in AWR session records
are metadata only — they never launch or configure the client.

Every supported client follows the same pattern:

1. Build both executables from an AWR checkout:

```sh
cargo build --locked -p awr-cli -p awr-mcp
```

2. Initialize the target project from its reviewed source manifest (see
   [Quickstart](quickstart.md)), then register the `awr-mcp` server with the client, pointing
   `--project` at the project's absolute path. Each server binds to one canonical project root;
   give servers for different projects distinct names.
3. Verify the connection — a config file on disk is not a live connection — and confirm project
   identity with a status read before any work.
4. Work the shared session lifecycle: start or resume a session, read context, checkpoint before
   handoff, end when you stop. See [CLI](cli.md) and [MCP tools](mcp.md).

The sections below give each client's merge paths, configuration, and caveats. Where a client
documents automatic lifecycle hooks, treat them as unverified until you have seen a real trigger and
receipt; the manual checkpoint/resume flow always works.

## Codex

Codex loads project configuration only for trusted projects; its CLI, desktop, and IDE clients share
MCP configuration on the same host. Pick one of two options: merge the MCP configuration template
into the project's `.codex/config.toml` (replacing its two paths, preserving existing
configuration), or register at user level:

```sh
codex mcp add awr -- /absolute/path/to/awr-mcp \
  --project /absolute/path/to/initialized/project
codex mcp get awr --json
```

In the client's `/mcp` view, confirm a connected `awr` server and verify the project identity before
work; a configured server is not proof of an active connection. Expected read tools:
`awr_project_status`, `awr_work_ready`, `awr_work_get`, `awr_context_compile`, `awr_search`.
Mutation tools: `awr_work_transition`, `awr_event_append`, `awr_evidence_record`. The server does
not require a model API key.

Codex is the one client with an installer for automatic lifecycle hooks: use `awr client install` to
preview the exact configuration before `--accept`. Installation preserves existing hooks and never
approves their trust automatically. Codex documents `SessionStart` (`startup|resume|clear|compact`),
`PreCompact` (`manual|auto`), and `SessionEnd` (advisory, short timeout). Verify actual trigger
delivery in the client — until then, checkpoint manually before handoff or compaction.

## Claude Code

Claude Code connects through a named controlled adapter (adapter id `claude_code`). It is not
auto-startable: AWR will not launch Claude Code for you, and it will not stop it on its own — those
steps stay with you. What the adapter does support is reading status, reconnect/resume, and result
forensics.

Operator path:

1. Start Claude Code yourself.
2. Bind it to AWR with the generic client and the native conversation ID:

```sh
awrj client bind --client generic --external-session claude:<native-id> \
  --work "$AWR_WORK" --session "$AWR_SESSION"
```

3. When you need AWR observations, report phases through the shared CLI's external execution
   report.
4. On retry, reconnect the same execution identity before starting anything new.

## Kimi

This guide targets Kimi Code 0.41.0 (inspected 2026-09-08); older `kimi-cli` releases use different
configuration paths and flags, so check `kimi --version` and `kimi --help` first. Use Kimi's
terminal tool to call the AWR CLI, and optionally connect the same project over stdio MCP by merging
this server into the project's `.kimi-code/mcp.json` (user level: `~/.kimi-code/mcp.json`),
preserving other entries:

```json
{
  "mcpServers": {
    "awr": { "command": "/absolute/path/to/awr-mcp",
      "args": ["--project", "/absolute/path/to/initialized/project"],
      "cwd": "/absolute/path/to/initialized/project" }
  }
}
```

Project configuration requires workspace trust. Use `/mcp-config` for configuration and `/mcp` to
inspect connection status; servers added by editing configuration join newly created sessions.
Confirm project identity with `awr_project_status`. The provider label for Kimi session metadata is
`moonshot`.

Kimi's own `kimi --continue`, `kimi --session <kimi-conversation-id>` and `/compact` operate on its
conversation; their IDs are separate from AWR IDs. After a compact, compile context for the same
active AWR session — an explicit AWR resume is only for a real handoff. Kimi documents
`SessionStart`, `SessionEnd`, `PreCompact`, and `PostCompact` hooks, but this integration installs
no automatic adapter; use the manual checkpoint process.

## Cursor

Checked against Cursor 3.18.9 (2026-09-17). Do not run `awr client install` for Cursor
(`Unsupported`), and do not pass `--client cursor` to bind (`InvalidInput`) — use the generic client
identity shown below. Merge the stdio template into the project `.cursor/mcp.json` or the user
`~/.cursor/mcp.json`:

```json
{

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [originoneai/awr](https://github.com/originoneai/awr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
