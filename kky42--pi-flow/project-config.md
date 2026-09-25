---
trigger: always_on
description: - Code must explain its own behavior through clear names, types, structure, and tests. Do not use comments to narrate what the next line, branch, loop, function, or assertion does.
---

# Agent Notes

## Comment standards

- Code must explain its own behavior through clear names, types, structure, and tests. Do not use comments to narrate what the next line, branch, loop, function, or assertion does.
- Keep a comment only when it preserves information that cannot be recovered from the code itself. Allowed cases are:
  - why a design was chosen and which tradeoff it accepts;
  - why a default, limit, timeout, or threshold has its particular value;
  - an external constraint, compatibility requirement, protocol rule, security boundary, or non-obvious failure mode;
  - a temporary workaround or compromise and the condition under which it can be removed;
  - an actionable `TODO`/`FIXME` with the missing behavior or blocking condition.
- Prefer improving unclear code over explaining it with a comment. Delete decorative section labels, comments that repeat symbol names or test assertions, and stale historical narration.
- Documentation for public APIs exposed through `package.json` exports may describe contracts, inputs, outputs, and failure semantics, but must not paraphrase the implementation. A TypeScript `export` alone does not justify documentation.
- When changing nearby code, update or remove its comments in the same change. A misleading comment is worse than no comment.
- These rules apply to source, tests, scripts, and examples. Do not edit generated output, vendored dependencies, or `refs/` solely to enforce them.

## pi-flow run_agent contract

- This repo implements a lightweight pi extension named `pi-flow`, not a fork of `refs/pi-subagents`.
- The registered tool is `run_agent`. v2 adds an opt-in `run_workflow` tool (see "pi-flow workflows (v2)" below); the v1 contract here still governs the `run_agent` tool.
- Tool parameters use `label`, `prompt`, optional `profile`, and optional `session_key` for a resumable child conversation.
- `label` is UI/routing metadata. `prompt` is the full subagent task.
- The only V1 built-in profile is `general-purpose`. There are no built-in aliases.
- `profile` defaults to `general-purpose`.
- `general-purpose` adds no role prompt.
- Do not replace pi's base system prompt in v1.
- Every top-level `run_agent` call follows Pi's normal Tool lifecycle: `execute()` remains pending until the child completes, fails, or is aborted, and then returns one terminal envelope containing `task_type`, `status`, `label`, and plain-text `content`, plus `session_key` only when a resumable child session actually started. There is no accepted Tool result, custom completion notification, polling, steering, scheduling, per-call model override, or per-call thinking override.
- Top-level calls use the Tool abort signal. Before session-tree navigation or shutdown, abort and drain active PiFlow calls, then reset task and session-key state without creating a model turn.
- Tool calls accept only `label`, `prompt`, optional `profile`, and optional `session_key`; backend/model/thinking selection is profile-based.
- Subagent timeout is a global operator-facing guardrail (`subagentTimeoutMs` / `--subagent-timeout-ms`), not a per-call parameter on `run_agent` or workflow `run_agent()`. The runtime timeout is owned by `spawnSubagent` after callers acquire a concurrency slot, so queue time does not count against it.
- Direct subagents start with a fresh persisted conversation and the same working directory when `session_key` is omitted. PiFlow generates and returns the effective key once the child starts, so a later call can resume it; failed calls that never started a child carry no key. A supplied unbound key names a new child; a bound key continues that child. Parent messages and tool results are not inherited. The extension maps the key to the backend-native session/thread id and persists the direct-subagent binding as parent-session custom state.
- Pi-backed subagents inherit the caller's current model and thinking level unless a custom profile pins `model` or `thinking`.
- Custom profiles may set `backend: pi` (default), `backend: codex`, or `backend: claude`. Every direct run_agent call is persistent from its first turn. Unkeyed Workflow-internal calls may remain one-shot. Codex uses `codex exec resume --json ... <session_id> -` for continuation; Claude uses `--resume <session_id>`. Both receive the task on stdin, profile prompt/model/thinking settings, bounded output handling, usage parsing, and the existing external CLI permission bypasses. Only use external backends in trusted repositories.
- `tools` frontmatter is a pi-backend child-session allowlist only. External CLI profiles use their CLI's own tool and permission surface.
- There is no pi-flow permissions system in v1. Profiles are ordinary agents with optional prompts and tool allow-lists; external backends are explicit user dependencies.
- Subagents cannot invoke PiFlow delegation tools. Pi-backed children receive neither `run_agent` nor `run_workflow` nor the flow prompt (`buildFlowPrompt`); external CLI children do not load the PiFlow extension.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kky42/pi-flow](https://github.com/kky42/pi-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
