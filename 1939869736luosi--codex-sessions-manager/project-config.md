---
trigger: always_on
description: Use this skill when the user wants to inspect, search, export, verify, clean up, delete, restore, or purge local Codex sessions stored under a Codex root such as ~/.codex, including auditing leftovers after Codex Desktop's built-in delete.
---


# Codex Sessions Manager

## Overview

This skill manages local Codex sessions through the `codex-sessions` toolkit.

Use it when the user wants to work with local Codex conversation history instead of the current live conversation.

Codex Desktop has a built-in delete action for archived chats. Use this toolkit when the user needs local proof of what remains, exact ID-based cleanup, batch handling, recoverable trash, restore, or post-delete verification.

This repository provides:

- a Node / TypeScript CLI
- a local stdio MCP server
- this Skill entrypoint

The project is not a UI product, TUI, detail page, incremental scanner, or automatic cleanup service.

## Setup

Install the CLI before using fallback commands:

```bash
npm install -g codex-sessions-manager
```

This provides:

```text
codex-sessions
codex-sessions-mcp
```

Check the installed version with `codex-sessions --version`. The MCP bin also supports `codex-sessions-mcp --version` for packaging or install verification without starting the stdio server.

For local development, build the repository first:

```bash
cd <path-to-codex-sessions-repo>
npm install
npm run build
```

The default Codex root is:

```text
~/.codex
```

Use `--root <path-to-codex-root>` when working with another Codex root.

## When To Use

Use this skill for requests like:

- "List my recent Codex sessions"
- "Find sessions for this project"
- "Show this session"
- "Find the session family for this session"
- "Show parent and side/fork child sessions"
- "Find side conversations for this session"
- "Audit what remains locally after the official Codex UI delete/archive action"
- "Find likely local residue without knowing the session ID"
- "Preview deleting likely root residue candidates"
- "Plan deleting these explicit session IDs without deleting anything"
- "Export this session"
- "Preview deleting these sessions"
- "Check what Codex Desktop left behind after deleting a chat"
- "Move these sessions to trash"
- "Restore this trash entry"
- "Purge this trash entry"
- "Verify whether this session is fully removed"
- "Inspect the Codex root before deleting or restoring"
- "Clean stale JSONL indexes"

Do not use this skill for:

- generic ChatGPT history questions
- replacing the ordinary Codex Desktop delete UI for simple archived-chat deletion
- non-Codex chat clients
- editing the current live conversation
- automatic cleanup schedules
- provider or model repair

## Preferred Order

### 1. Prefer MCP first

If the `codex-sessions` MCP server is available in the current agent session, use these tools:

- `inspect_root`
- `list_sessions`
- `summarize_sources` (read-only source summary)
- `list_projects`
- `get_session`
- `get_session_family` (read-only session family inspection)
- `audit_session` (read-only residue audit)
- `audit_root` (read-only root residue scan; candidates are not deletion recommendations)
- `preview_root_delete` (read-only root delete preview; never deletes and never recommends deletion)
- `export_session_backup`
- `preview_delete_sessions`
- `plan_delete_sessions` (read-only delete planning; cannot execute deletion)
- `preview_delete_plan` (read-only plan-file / inline-plan stale check; cannot execute deletion)
- `delete_sessions` (without `confirm=true`, returns preview only; with `confirm=true`, executes after the caller has reviewed the intended scope; pass `trash=true` for recoverable deletion; P11 exact-key global-state refs follow the same preview/confirm safety model)
- `list_trash`
- `restore_sessions` (requires `confirm=true`)
- `purge_trash` (requires `confirm=true`)
- `cleanup_session_indexes` (requires `confirm=true` to rewrite JSONL indexes)
- `cleanup_stale_indexes` (requires `confirm=true` to rewrite JSONL indexes)
- `verify_sessions`

Use MCP tools first. CLI is the fallback when MCP is unavailable or blocked.

For session lookup, narrow in this order:

1. project
2. status
3. updated / created time
4. preview or `get_session`

For project-aware listing, pass `project` to `list_sessions` or use `groupBy="project"`.

For time filters, pass `updatedAfter`, `updatedBefore`, `createdAfter`, or `createdBefore`. Date-only filters use the local calendar day. Timezone-less datetime strings must be rejected.

For source-aware listing, pass `sourceKind`, `source`, `threadSource`, `agentRole`, `agentNickname`, `modelProvider`, or `model` to `list_sessions`. Use `summarize_sources` for a read-only count by `sourceKind`, raw `source`, `thread_source`, `model_provider`, `model`, and `agent_role`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [1939869736luosi/codex-sessions-manager](https://github.com/1939869736luosi/codex-sessions-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
