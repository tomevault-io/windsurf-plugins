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

## pi-flow Agent contract (v1)

- This repo implements a lightweight pi extension named `pi-flow`, not a fork of `refs/pi-subagents`.
- The registered tool is `Agent`. v2 adds an opt-in `workflow` tool (see "pi-flow workflows (v2)" below); the v1 contract here still governs the `Agent` tool.
- Tool parameters follow the Claude Code-style shape: `description`, `prompt`, optional `subagent_type`, and optional `session_key` for an explicitly resumable child conversation.
- `description` is UI/routing metadata. `prompt` is the full subagent task.
- The only V1 built-in profile is `general-purpose`. There are no built-in aliases.
- `subagent_type` defaults to `general-purpose`.
- `general-purpose` adds no role prompt.
- Do not replace pi's base system prompt in v1.
- V1 is foreground-only. Do not add background execution, result polling, steering, scheduling, per-call model override, or per-call thinking override. Session continuation is explicit and foreground-only via caller-chosen `session_key`.
- Tool calls still only accept `description`, `prompt`, optional `subagent_type`, and optional `session_key`; backend/model/thinking selection is profile-based.
- Subagent timeout is a global operator-facing guardrail (`subagentTimeoutMs` / `--subagent-timeout-ms`), not a per-call Agent or workflow `agent()` parameter. The runtime timeout is owned by `spawnSubagent` after callers acquire a concurrency slot, so queue time does not count against it.
- Subagents start with a fresh one-shot conversation and the same working directory when `session_key` is omitted. Parent conversation messages and tool results are not inherited. Passing the same caller-chosen `session_key` creates/continues that child backend conversation instead. The extension maps `session_key` to the backend-native session/thread id internally and persists the direct-Agent mapping as parent-session custom state.
- Pi-backed subagents inherit the caller's current model and thinking level unless a custom profile pins `model` or `thinking`.
- Custom profiles may set `backend: pi` (default), `backend: codex`, or `backend: claude`. Codex-backed profiles run external `codex exec --json --dangerously-bypass-approvals-and-sandbox --ephemeral -- -` for one-shot calls, omit `--ephemeral` for keyed first calls, and use `codex exec resume --json ... <session_id> -` for keyed continuation; they send the task prompt on stdin, pass the profile body as `developer_instructions`, pass profile `model`/`thinking` through Codex CLI, parse `thread.started.thread_id`, token usage from Codex JSONL events, and estimate cost for listed models. Claude-backed profiles run external `claude -p --output-format stream-json --verbose --dangerously-skip-permissions --no-session-persistence` for one-shot calls, omit `--no-session-persistence` for keyed first calls, add `--resume <session_id>` for keyed continuation, send the task prompt on stdin, pass the profile body as `--append-system-prompt`, pass profile `model`/`thinking` through Claude Code, parse `system/init.session_id`, parse token usage from stream JSON, and use Claude Code's reported `total_cost_usd` when available. External CLI backends intentionally run in yolo/no-approval mode; only use them in trusted repositories.
- `tools` frontmatter is a pi-backend child-session allowlist only. External CLI profiles use their CLI's own tool and permission surface.
- There is no pi-flow permissions system in v1. Profiles are ordinary agents with optional prompts and tool allow-lists; external backends are explicit user dependencies.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kky42/pi-flow](https://github.com/kky42/pi-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
