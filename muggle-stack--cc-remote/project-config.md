---
trigger: always_on
description: handles this structurally: one `async for` per turn runs through the interrupt
---

# AGENTS.md — cc-remote

Guidance for Codex (and human contributors) working in this repo. User-facing setup/run docs live in [README.md](README.md) / [README_en.md](README_en.md).

## What this is
Self-hosted remote control for Claude Code and Codex: a phone/browser drives a
local `claude` or `codex` session through a WebSocket relay. Two independent links:
- **model link** — the local CLI → whatever its own settings/authentication point
  at. cc-remote never touches model credentials or the model API.
- **control link** (this repo) — client ⇄ relay ⇄ wrapper ⇄ Claude Agent SDK /
  Codex app-server. Native CLI ownership is detected and mirrored separately.

## Deployment

The repository skill is
[`.agents/skills/cc-remote-deploy/SKILL.md`](.agents/skills/cc-remote-deploy/SKILL.md).
Use it for deployment, upgrade, verification and recovery requests; agents that
do not auto-discover repository skills must read it explicitly.

The repository-owned deployment procedure is [`deploy/README.md`](deploy/README.md).
Read its automation contract and the relevant installation path completely
before any deploy, redeploy, recovery, verification, or rollback. Do not depend
on out-of-tree instructions, and do not add an operator's host aliases,
usernames, IPs, domains, home directories, or credentials to this repository.
Resolve that inventory from the environment the operator placed in scope.

Use one tested source snapshot or one set of release artifacts for all protocol
tiers, stage and validate before touching live services, preserve external
configuration and private state, and use the repository's immutable activation
transactions. A dropped SSH/control connection is an unknown result: inspect
the original transaction and live state before deciding whether a retry is
safe. Deployment is complete only after protocol/build identity, service
stability, public health, and expected Wrapper connectivity are verified.
For Codex Code, also follow `deploy/README.md`'s shared-control-plane acceptance:
verify each account's daily CLI and Wrapper connect to the same official
app-server, not a private stdio fallback. Do not force takeover or kill a live
CLI to satisfy deployment checks.
After core deployment checks, follow the optional Codex App checkpoint in that
guide: detect an installed App on an in-scope desktop, ask before attaching it,
and keep a decline or pending answer separate from deployment success. App
attachment and optional App-control MCP tools are separate user choices.

## Critical constraints / traps
- **Drain footgun**: after `ClaudeSDKClient.interrupt()`, the SDK does NOT kill
  the session — the current turn's stream still emits a terminal
  `ResultMessage(subtype="error_during_execution")`. You MUST keep consuming
  `receive_response()` until that ResultMessage before the next `query()`, else
  stale deltas from the interrupted turn bleed into the new turn. The wrapper
  handles this structurally: one `async for` per turn runs through the interrupt
  to the terminal ResultMessage; state only returns to `idle` (and the next
  query is only accepted) after that break. Reject-while-busy prevents a second
  query racing the drain.
- **cwd must match resume**: a session's jsonl lives at
  `~/.claude/projects/<cwd-with-/-as->/<uuid>.jsonl`. `ClaudeAgentOptions.cwd`
  MUST equal the original session's cwd or `resume` can't find it.
- **SDK pinned to `claude-agent-sdk==0.2.151`**: message-type shapes and the
  interrupt/drain contract can shift between patch versions. Re-run the
  interrupt+drain verification after any upgrade (`SdkHandle.preflight()` guards
  the exact verified patch at startup).
- **Claude Code is the user's daily CLI, not the SDK bundle**: Claude Code
  `>=2.1.258` is required and checked before a Claude session starts. The wrapper
  defaults `CLAUDE_BIN` to `~/.local/bin/claude` and passes that path explicitly
  to the SDK. An empty value keeps this default; only another absolute path may
  override it. Keep that CLI updated and signed in before starting the wrapper.
- **`include_partial_messages`** is a `ClaudeAgentOptions` field (set at
  construction, not on `query()`). Streaming events arrive as `StreamEvent`
  (`.event` = raw Anthropic API stream-event dict) — NOT
  `SDKPartialAssistantMessage` (doesn't exist in 0.2.151). Extract
  `content_block_delta` → `delta.text` from `StreamEvent.event`.
- **tool_use is batched, not streamed**: emit one `tool_use` event from the
  assembled `AssistantMessage` (full `input`), never as JSON-fragment deltas.
  Text deltas still stream live via `StreamEvent`.
- **Claude only — don't set `setting_sources=[]` for Code**: legacy single-account
  Code intentionally loads `~/.claude/settings.json`. Explicit account profiles
  keep the real HOME, clear ambient account selectors, and use
  `setting_sources=["user"]`. A profile rooted at the real per-user `~/.claude`
  must leave `CLAUDE_CONFIG_DIR` unset, retaining `~/.claude.json` and native
  keychain identity; other roots set it explicitly. Setting it to `~/.claude`
  changes the account file to `~/.claude/.claude.json`. Project/local settings
  may contain provider env or
  auth helpers and must not participate in an account-isolated child. Never pass

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [muggle-stack/cc-remote](https://github.com/muggle-stack/cc-remote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
