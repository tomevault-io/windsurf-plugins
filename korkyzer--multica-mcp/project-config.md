---
trigger: always_on
description: This guide documents patterns and best practices for orchestrating Multica through the MCP server, from an LLM chat (Claude Desktop, Codex Desktop, Claude Code, or any MCP-capable client).
---

# Multica MCP — Advanced Usage Guide

This guide documents patterns and best practices for orchestrating Multica through the MCP server, from an LLM chat (Claude Desktop, Codex Desktop, Claude Code, or any MCP-capable client).

It is the companion document to the [README](README.md). Start there for installation; come here when you want to get the most out of MCP-driven delegation.

Agent names in this document (`claude-sonnet`, `codex-standard`, `gemini-pro`, etc.) are examples. The actual names in your workspace are whatever you configured. Use `multica_list_agents` to discover them.

## Contents

1. [When to use the MCP](#1-when-to-use-the-mcp)
2. [Core workflow](#2-core-workflow)
3. [Safe issue creation](#3-safe-issue-creation)
4. [Writing issues that run on the first try](#4-writing-issues-that-run-on-the-first-try)
5. [Splitting large work](#5-splitting-large-work)
6. [Sequencing chains of issues](#6-sequencing-chains-of-issues)
7. [Dual reviewer pattern (PQR)](#7-dual-reviewer-pattern-pqr)
8. [Follow-up and correction](#8-follow-up-and-correction)
9. [Verification](#9-verification)
10. [Routing matrix](#10-routing-matrix)
11. [Tool-call caps](#11-tool-call-caps)
12. [Token optimization](#12-token-optimization)
13. [Known limitations](#13-known-limitations)
14. [Anti-patterns](#14-anti-patterns)
15. [Issue templates](#15-issue-templates)
16. [Safety notes](#16-safety-notes)

---

## 1. When to use the MCP

Trigger an MCP-driven Multica flow when one or more of these signals are present:

- The user wants to delegate, assign, triage, parallelize, or supervise work
- The request is larger than one focused coding session
- The work naturally splits into multiple independent tracks
- The user wants a backlog, sprint plan, sub-issues, or task routing
- The work needs a different model profile than the current chat
- The task is mostly bulk analysis, audit, review, or repository scanning
- The user asks which agent should handle a task
- The user wants to push more context to an agent already running in Multica
- The user wants to know which billed model actually ran

Do not trigger for trivial one-shot work that is faster to do locally than to delegate.

## 2. Core workflow

### 2.1 Check readiness

1. Call `multica_list_agents`
2. Call `multica_list_projects` if project placement matters
3. Match by current agent names, never by hardcoded IDs

If the user already named an agent, still confirm it exists in `multica_list_agents`.

### 2.2 Decide whether to delegate

Delegate when at least one is true:

- The work is parallelizable
- The work benefits from a different provider or context window
- The user mainly needs coordination rather than direct hands-on edits
- The current conversation would lose momentum if it tried to absorb all the work inline

Keep work local when:

- The next step is tiny and unblockable
- The answer is mostly explanation, not execution
- The task is urgent and the very next action depends on a result you can produce faster yourself

### 2.3 Pick the agent

Use the [routing matrix](#10-routing-matrix) as the default. **Consider context, not just task type.** A task resuming after a crash with partial work may be short enough for a cheaper agent; a fresh architectural investigation should never start on a fast/cheap agent — the reasoning window is insufficient.

## 3. Safe issue creation

The Multica daemon picks up issues whose status is `todo` **and** that have an assignee. To avoid an issue being picked up before it is ready:

```
ALWAYS create backlog-destined issues like this:
1. Create the issue WITHOUT an assignee (arrives in `todo`)
2. Move it to `backlog`: update_issue(status="backlog")
3. THEN add the assignee: update_issue(assignee="<agent>")
Backlog + assignee = safe. The daemon only picks up `todo` + assignee.
```

Via MCP: `multica_create_issue` without `assignee`, then `multica_update_issue` for status, then a second `multica_update_issue` for assignee.

## 4. Writing issues that run on the first try

Always include:

- A precise title with the expected outcome
- Enough description for the agent to act without guessing
- Repo or product context
- Explicit success criteria
- Constraints, non-goals, and output format
- The working directory if it matters (`cwd` parameter — it is injected as a markdown hint; the agent must still `cd` manually)
- A hard cap on `tool_calls` (see [caps table](#11-tool-call-caps))

**Never include a step that asks the agent to restart, stop, or otherwise manipulate the Multica daemon.** The agent runs inside the daemon and would kill its own parent process.

Use the [issue templates](#15-issue-templates) rather than improvising vague prompts.

### Disambiguate Claude Code vs Claude Desktop vs Codex in briefs

The configuration files and capabilities differ:

- Claude Code: `~/.claude.json` (MCP servers), `~/.claude/settings.json` (hooks), skills
- Claude Desktop: `~/Library/Application Support/Claude/claude_desktop_config.json` (MCP servers only)
- Codex CLI / Codex Desktop: `~/.codex/config.toml` (MCP servers; Codex Desktop reuses the CLI config)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Korkyzer/multica-mcp](https://github.com/Korkyzer/multica-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
