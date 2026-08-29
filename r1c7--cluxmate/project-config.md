---
trigger: always_on
description: - CluxMate = one Python agent core (`cluxmate/`) driving three front-ends: headless CLI (`cluxmate -p "..."`), Textual TUI (`cluxmate`), and an Electron desktop (`desktop/`) that drives the core over JSON-RPC stdio (`cluxmate agent stdio`). Providers are OpenAI-compatible only (`core/providers/` = base + openai + factory; no Anthropic provider).
---

# Project memory

## What CluxMate is

- CluxMate = one Python agent core (`cluxmate/`) driving three front-ends: headless CLI (`cluxmate -p "..."`), Textual TUI (`cluxmate`), and an Electron desktop (`desktop/`) that drives the core over JSON-RPC stdio (`cluxmate agent stdio`). Providers are OpenAI-compatible only (`core/providers/` = base + openai + factory; no Anthropic provider).
- Four risk modes (`plan` / `default` / `acceptEdits` / `yolo`): `plan` registers only read tools (no bash/write/delete/task/update_memory/MCP — hard isolation, nothing to approve); `default` approves `safe` automatically, asks for `write`/`dangerous`; `acceptEdits` auto-approves `write`; `yolo` disarms the sandbox boundaries entirely (the one explicit opt-out). Mode is per-session only, never persisted (permissions.json stores only `always_allow_tools`).
- The repo root IS a git repo (`.git/`, branch `master`, remote `git@github.com:r1c7/CluxMate.git`, init 2026-08-25, actively committed since). `.gitignore` excludes `docs/`, `scripts/`, `.cluxmate/`; `snapshots/*.png` (UI screenshots) ARE tracked. Checkpoint shadow-git repos live separately in `~/.cluxmate/checkpoints/<sha1(cwd)>.git` (see `core/checkpoints.py`) and never touch the user's real repo (GIT_DIR/GIT_WORK_TREE set via env + `GIT_CONFIG_GLOBAL=/dev/null`).

## Architecture — core invariants

- The session log is **event-sourced and the source of truth** (`cluxmate/core/session_log.py` docstring): provider message history is *derived* from the append-only log, never stored separately. Model-visible request == `[system from fold_request_header] + derive_messages()`; `derive_messages()` deliberately excludes the system prompt. `request/header` events record the full non-history envelope (config incl. model/max_tokens/context_window/mode/reasoning_effort + system prompt + tool schemas) with reason `initial`/`change`; only changed headers are appended (`header_equals`/`canonical_header` in `core/agent.py`). Compaction rewrites a *surface* projection via `replace` ops without erasing the underlying events, preserving a cache-stable prefix.
- Environment context — memory, skills, mode — is injected as **tagged synthetic `user/message` events** (`source: memory` / `skill` / `mode`; also `human`, `compaction`, `interruption`), NOT baked into the system prompt, so request prefixes stay prompt-cache stable. A mode or tool-schema change appends a fresh `request/header` with reason `change`.
- `AgentBuilder` (fluent, `core/builder.py`) is the single wiring point: it registers tools, the WriteFence, the bash/MCP sandbox, and subagent types. A `chat/set_mode` change rebuilds the agent so a mode switch re-arms/disarms sandbox boundaries.
- Subagents are child `SessionLog`s linked to the parent via `subagent/spawn` events; replay walks the whole delegation tree depth-first (`session_log_store.py`). Recursion capped at `MAX_SUBAGENT_DEPTH = 4` (root = depth 0); at the cap the `task` tool is withheld. `general-purpose` children may recurse to either type; `explore` children recurse only within `explore` (TaskTool allowlist keeps the chain read-only).
- Torn session tails are dropped on load; an interrupted turn is closed with synthetic `tool/result` + `turn/end {interrupted}` (crash-repair closers, dropped when premature); undo rewinds to a turn boundary via a single `truncate`.

## Security: sandbox (READ the docs before touching this code)

Two enforcement boundaries sit behind the permissions policy (`core/permissions.py` decides what is *allowed*; the boundaries make denials stick). Authoritative Chinese docs: `docs/plans/sandbox-current-state.md` (current implementation) and `sandbox-threat-model.md` (decisions/threat model). Read them before touching `tools/_fence.py` / `tools/_sandbox.py` / `core/grants.py`.

- **WriteFence (T1, in-process)** — guards the 5 file tools `write_file` / `search_replace` / `multi_edit` / `multi_write` / `delete_file` (batch tools check each item; out-of-bounds items are skipped). Canonicalize-then-contain: `resolve(strict=False)` (resolves `..` and symlinks) → deny check → contain check, before any I/O.
  - Writable roots defined in exactly one place: `WriteFence.roots()` = session cwd + platform temp dir + exactly `~/.cluxmate/AGENTS.md` (whitelisted so `update_memory`'s "edit global entries with search_replace" contract works) + user-granted folders. The rest of `~/.cluxmate` (config.json with API keys, session logs, checkpoints) is off-limits.
  - `<cwd>/.cluxmate/` is a **deny subtree** (takes precedence over roots): it holds CluxMate's privileged project state (permissions.json always-allow, mcp.json which spawns subprocesses on load, skills.json) — a prompt-injected model must not edit its own permission config. Deny-subtree rejections are NOT openable by escalation.
  - Enabled in all modes except `yolo`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [r1c7/CluxMate](https://github.com/r1c7/CluxMate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
