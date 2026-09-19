---
trigger: always_on
description: > **Repo note (Godot MCP fork):** the canonical GDScript plugin lives in the
---

# AGENTS.md - Godot AI

> **Repo note (Godot MCP fork):** the canonical GDScript plugin lives in the
> root `addons/godot_ai/` and `addons/godot_omni/` folders — the release
> archives are built from them. The duplicate `plugin/addons/` tree mentioned
> in the upstream sections below was **removed in v5.0.2**; treat any
> reference to it as stale.

This guide is for any AI assistant working in this repository. Keep Claude-specific files such as `.claude/CLAUDE.md` and `.claude/skills/*` as thin pointers to this shared guidance.

## PR activity monitoring

Whenever a PR is created, opened, or identified in the conversation, check its
state and start monitoring without asking again, unless it is already closed
or merged. This authorizes read-only monitoring and notifications, not posting
comments, changing code, rerunning CI, or merging.

- Discover the current environment's tools. Prefer a native PR activity
  subscription (such as `subscribe_pr_activity`) when actually available;
  confirm success and retain its subscription identifier. Do not assume a
  particular tool namespace or that GitHub access includes subscriptions.
- If subscriptions are unavailable but durable task automations are supported,
  reuse an existing monitor for the same repository and PR or create a
  15-minute heartbeat attached to the current task. Poll PR comments, inline
  review comments, submitted reviews, CI results, merge conflicts, and PR state
  using the authenticated GitHub connector or `gh`. Keep the last seen state,
  report existing actionable feedback once, and notify only on new actionable
  feedback, failures, conflicts, access failures, or closure/merge. Stay quiet
  otherwise; stop the monitor after closure/merge.
- If neither capability is available, check current feedback and CI, then
  explicitly report that persistent monitoring was not started and which
  capability is missing. A one-time read, shell polling process, GitHub email
  subscription, or written instruction is not a durable task subscription.
- Report the confirmed monitoring mechanism when starting it. Never claim
  monitoring is active without a successful subscription/automation response.

This policy applies to local and cloud sessions. Check capabilities separately
in each environment: local Codex settings and credentials do not establish
cloud tool availability. Do not add a guessed MCP endpoint or install an
integration merely because it has GitHub in its name. A future provider must
be verified to expose the subscription and deliver events to the task.

## What this project is

A production-grade MCP server for Godot. Python server (FastMCP v3) communicates over WebSocket with a GDScript editor plugin. AI clients call MCP tools → Python routes commands → Godot plugin executes against the editor API → results flow back.

## Architecture

- **Protocol**: v4 protocol 2 uses mutual, transcript-bound HMAC over a private per-backend WebSocket capability. The editor reveals metadata only after verifying the server proof; the server publishes a reserved peer only after its final ACK succeeds. Missing/wrong capabilities, protocol-1 frames, duplicate keys, and downgrade attempts fail closed.
- **Transport boundary**: WebSocket remains loopback-only and HTTP is localhost-first, but neither relies on loopback as authentication. HTTP/status/lease require a separate bearer capability; both transports have finite connection/body/frame/session budgets. Never add tokenless, bare-URL, or legacy-handshake fallback. Full contract: [docs/plugin-architecture.md](docs/plugin-architecture.md#security-model).
- **Session model**: Multiple Godot editors connect through one authoritative `_entries` table. Public `Session` values are immutable snapshots; settlement/removal is bound to the exact peer and request reservation. Tools route through the active or explicitly pinned session.
- **Handler/Runtime layer**: Shared handlers in `src/godot_ai/handlers/` contain tool logic. They depend on `DirectRuntime`, the in-process runtime adapter. Tools and resources are thin wrappers that create a runtime and delegate.
- **Readiness gating**: writes check session readiness before executing — Python write handlers must `await require_writable_async()` (`handlers/_readiness.py`), and every new plugin response builder must stamp the envelope-level `readiness` field. `EDITOR_NOT_READY` is frozen as a top-level code; never promote a `data.sub_code` into `error.code`. Self-healing, the importing-hold, and the sub-code vocabulary: [docs/plugin-architecture.md](docs/plugin-architecture.md#session-and-readiness-model).

## Project structure

Read the tree directly — `src/godot_ai/` (Python MCP server) and
`plugin/addons/godot_ai/` (GDScript editor plugin) are the two roots, and the
directory names say what they hold. The parts the layout does *not* tell you:

- `plugin/addons/godot_ai/` is the canonical GDScript copy. `test_project/addons/godot_ai`
  is a locally-built symlink (Windows junction) into it, not tracked in git.
- `src/godot_ai/tools/_meta_tool.py` holds `register_manage_tool`, the rollup factory.
- `src/godot_ai/middleware/` registration order is load-bearing — see "Key conventions".

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bebabinlarsson-blip/Godot-MCP](https://github.com/bebabinlarsson-blip/Godot-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
