---
trigger: always_on
description: This repository, **点将台**, implements a local **STDIO MCP server** that allows Codex to
---

# AGENTS.md

## Project purpose

This repository, **点将台**, implements a local **STDIO MCP server** that allows Codex to
delegate already-scoped coding tasks to Claude Code through the Claude Agent SDK.

Read `docs/CODEX_CLAUDE_AGENT_MCP_SPEC.md`
before making architectural changes.

## Non-negotiable architecture

- Codex is the intelligent orchestrator.
- This MCP server is a thin, deterministic job runner and session manager.
- Do not add semantic task decomposition inside the MCP server.
- Do not let the MCP server invent acceptance criteria.
- Task, acceptance criteria, resolved cwd, and execution session must stay bound
  to the same `job_id`; reject review/resume mismatches.
- Execution and review are different Claude sessions.
- A reviewer must not resume the execution session.
- A reviewer must not modify the project under review (read-only by default).
- A failed review may be sent back to the original execution session via resume.
- `review=true` with empty acceptance criteria is an input error.
- `review=false` must never trigger a hidden review call.
- Do not silently add Claude calls that the caller did not request.

## STDIO transport

- Use STDIO MCP transport only. No HTTP server.
- A launched MCP server process must handle multiple tool calls over the same
  stdin/stdout connection.
- Do not start a new MCP server for each tool call.
- Do not assume there is globally only one MCP server process across all Codex
  threads/sessions.
- The implementation must remain correct when several copies run at once.

## stdout/stderr discipline (mandatory)

- stdout is reserved exclusively for MCP protocol messages (newline-delimited
  JSON-RPC).
- logs, diagnostics, startup messages, warnings, and tracebacks go to stderr.
- never use ordinary `print()` to stdout for debugging.
- never pipe raw Claude CLI output directly to MCP stdout.
- capture Claude results and return them as structured MCP tool results.

## Process model

- One started STDIO MCP server process may serve many tool calls during its
  connection lifetime.
- Claude Agent SDK sessions are separate execution sessions and may create their
  own Claude CLI subprocesses.
- Concurrency inside one MCP server instance is controlled with an
  application-level async semaphore/queue (`scheduler.py`).
- Cross-process global concurrency is NOT guaranteed by a per-process semaphore.

## Claude Agent SDK

- Use the current supported `query()` API (`claude_agent_sdk.query`).
- Explicitly use the Claude Code system prompt preset for coding execution:
  `system_prompt={"type": "preset", "preset": "claude_code", "append": ...}`.
- Preserve and return execution `session_id`.
- Use SDK `resume=<session_id>` for `continue_task`.
- Create a fresh session for review (no `resume`).
- Configure permissions explicitly via `allowed_tools` / `disallowed_tools` /
  `permission_mode`. Do not default to `bypassPermissions`.
- Use Agent SDK JSON Schema structured output and validate the returned payload.
- Execution: read/search/edit/bash tools allowed; it owns necessary validation.
  For unclear Bugs it reproduces before editing and retests the same condition;
  a reliable failing test or clear error evidence already counts. Review stays
  Read/Grep/Glob-only and names missing targeted evidence instead of running it.

## API behavior

Core tools: `execute_task`, `review_task`, `continue_task`, `run_job`,
`run_jobs`, and `get_job_status`, plus `ping` for liveness. Keep inputs and outputs structured
(Pydantic models). Do not return full transcripts to Codex.

## Review behavior

Reviewer instructions require independent inspection. Reviewer result is one of
`PASS` / `FAIL` / `FAILED`. On `FAIL`, return unmet criteria and concrete
evidence. Review checks code/logic and reported validation without repairing or
running tests; insufficient evidence is a concrete FAIL for CC-1 to address.

## State

Use explicit state transitions. Persist at least `job_id`,
`execution_session_id`, `review_session_id`, `cwd`, `project_root`, `status`,
timestamps in SQLite (`session_store.py`); schema changes must stay additively
backward compatible for existing databases. Multiple MCP server processes may exist, so DB
access must tolerate multiple processes: WAL, `busy_timeout`, `BEGIN IMMEDIATE`,
`UNIQUE(job_id)`. Review/resume state claims must use atomic compare-and-set.
Client cancellation must leave the affected stage as `INCOMPLETE`, persist the
aggregate `EXECUTION_INCOMPLETE` or `REVIEW_INCOMPLETE`, and attach a structured
`CANCELLED` error. A preallocated session becomes resumable only after the SDK
stream confirms the same session id.

## Concurrency

- One Claude concurrency limit per MCP server process in v1.
- Execution and review both count against that limit.
- Queue excess jobs rather than starting nested MCP servers.
- Do not attempt semantic file-conflict resolution in v1.
- Upstream Codex must not parallelize overlapping write tasks.

## Security

- Accept an optional explicit `project_root` (`execute_task`, `review_task`,
  `run_job`, each `run_jobs` `JobSpec`) supplied by upstream Codex. Never infer
  it from the MCP process cwd. When supplied it must be an existing absolute

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zjgxkj/codex-cc-orchestrator](https://github.com/zjgxkj/codex-cc-orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
