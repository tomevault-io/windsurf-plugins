---
trigger: always_on
description: This repository implements **ClaudeForClaude (CLFC)**, a Windows-first session indexer and workspace helper for Claude Code.
---

# AGENTS.md

## Purpose

This repository implements **ClaudeForClaude (CLFC)**, a Windows-first session indexer and workspace helper for Claude Code.

CLFC treats Claude Code transcript JSONL files as the durable source of session history and builds a lightweight local control plane on top of them.

See `docs/claude-conversation-system.md` for observed transcript structure and implementation implications.

The project is intentionally parallel to CodexForCodex, but Claude Code has different primitives:

- Claude Code uses `CLAUDE.md`, `.claude/`, settings, hooks, agents, skills, and prompt flags.
- Claude Code stores plaintext transcripts under `~/.claude/projects/...`.
- Claude Code resumes sessions through the `claude` CLI, not a Codex app-server.
- Claude Code can append or replace system-prompt text per invocation, but its internal default system prompt is not published and should not be reconstructed.

### Local Claude Code backend

This workspace is expected to run Claude Code through Ollama's Anthropic-compatible endpoint when the user's global Claude Code settings are configured that way:

```text
ANTHROPIC_BASE_URL=http://localhost:11434
ANTHROPIC_AUTH_TOKEN=ollama
```

The current preferred cloud-backed Ollama model is:

```text
gpt-oss:120b-cloud
```

The smaller background/default fast model may be:

```text
gpt-oss:20b-cloud
```

Do not assume Anthropic-hosted Claude models are available unless the user asks to switch back or provides Anthropic authentication. When debugging Claude Code execution, first check whether requests are being routed to Ollama through `ANTHROPIC_BASE_URL`.

---

## Core model

### Claude Code transcript

A Claude Code session recorded under:

```text
%USERPROFILE%\.claude\projects\<project>\<session>.jsonl
```

If `CLAUDE_CONFIG_DIR` is set, it replaces `%USERPROFILE%\.claude` as the Claude config root.

Each transcript may contain top-level metadata such as:

- `sessionId`
- `cwd`
- `timestamp`
- `version`
- message records
- tool call and tool result records

Transcripts are plaintext. Anything Claude Code reads, receives, or prints through tools can land in the transcript. CLFC must never copy full transcript content into its index.

### CLFC session

A CLFC-managed wrapper around a Claude Code session.

Stored metadata includes:

- `session_id`
- `session_name`
- `claude_session_id`
- `cwd`
- `workspace_hash`
- `project_key`
- `created_at`
- optional `display_name`
- optional `preview`
- optional `updated_at`
- optional `transcript_path`
- optional `source_session_id`
- optional `source`
- optional `claude_version`
- optional `model`
- optional `effort`
- optional `permission_mode`
- `session_kind` (`normal | template | worker`)
- optional `template_session_id`
- optional `worker_state` (`idle | leased | done | failed`)

The `preview` field, when present, must be short and user-message-only. Do not index tool outputs, file contents, command output, pasted secrets, or large message bodies.

### Workspace

The current working directory where CLFC is initialized.

Each workspace has:

- local metadata under `.clfc/`
- its own `workspace_hash`
- its own active session pointer
- its own transcript-list cache
- per-session runtime workspaces under `~/.clfc/<workspace_hash>/<session_name-or-claude_session_id>/`

CLFC distinguishes two directories:

- the **real workspace**, where the user's project files live
- the **session workspace**, where CLFC runs `claude` so it can control session-local `CLAUDE.md`, prompt files, settings overlays, and runtime metadata

When executing Claude Code from the session workspace, CLFC grants access to the real workspace with `--add-dir <real_workspace>`.

---

## Storage layout

### Local workspace metadata

```text
.clfc/
  workspace.json
  cache/
    transcript_list.json
```

### Global CLFC data

Windows default location:

```text
%LOCALAPPDATA%\clfc\
  state.json
  workspaces\
    <workspace_hash>\
      <claude_session_id>.json
```

`state.json` stores global fetch state such as `last_search_time`.

`workspaces/<workspace_hash>/<claude_session_id>.json` stores one indexed Claude Code session record per file.

### Claude Code source data

Windows default location:

```text
%USERPROFILE%\.claude\
  projects\
    <project>\
      <session>.jsonl
```

`clfc fetch` scans this tree directly. If `CLAUDE_CONFIG_DIR` is set, scan `<CLAUDE_CONFIG_DIR>\projects` instead.

### Per-session runtime workspace

```text
%USERPROFILE%\.clfc\
  <workspace_hash>\
    <session_name-or-claude_session_id>\
      session.json
      CLAUDE.md
      system_prompt.md
      settings.json
```

The files above are CLFC-owned copies or overlays. They are not the user's project files.

---

## CLI model

The current implemented slice is intentionally smaller than the long-term command set:

```bash
clfc doctor
clfc init
clfc add <display_name>
clfc add <display_name> <session_id_or_prefix>
clfc add <display_name> --checkout
clfc interactive
clfc interactive --dangerously-skip-permissions
clfc exec "<prompt>"
clfc exec --template <path> --var key=value
clfc exec --fork --checkout-new --display-name <display_name> "<prompt>"
clfc resume <session_id_or_prefix>
clfc resume

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Longseabear/ClaudeForClaude](https://github.com/Longseabear/ClaudeForClaude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
