---
trigger: always_on
description: Build the best combined coding-agent experience from Codex and Claude Code, with a strong focus on simplicity, thoughtful defaults, and a polished user experience. Prioritize important, widely useful workflows over obscure features or exhaustive parity.
---

# Agent Instructions

## Product direction

Build the best combined coding-agent experience from Codex and Claude Code, with a strong focus on simplicity, thoughtful defaults, and a polished user experience. Prioritize important, widely useful workflows over obscure features or exhaustive parity.

## Deliberate non-goals

Do not implement a dedicated Plan mode, Vim or other modal editing modes, Jupyter notebook parsing or editing, durable command allow/deny history, dedicated IDE integrations, a separate Rig login flow, or niche compatibility features whose primary value is exhaustive upstream parity. Rig uses the credentials managed by the system Codex and Claude Code installations, so users should sign in through those assistants instead. Planning should remain part of the normal agent workflow. Auto permissions should review the current action and user authorization without learning a persistent command-execution policy. Skills should follow Codex behavior and scope, not Claude Code's expanded skill runtime. Only reconsider these boundaries when the user explicitly changes the product direction.

## Permissions and security

Rig has one permission model for every provider. Codex, Claude, Pi, Grok, MCP, and future tool surfaces must execute through the same `AgentContext`, filesystem boundary, shell sandbox, and `PermissionContext`. Provider differences belong in tool names, argument schemas, result formatting, and model guidance; they must not create provider-specific security paths in the agent loop.

The permission modes are:

- Read only permits inspection and non-mutating commands while blocking workspace changes, shell network access, and sensitive private reads.
- Workspace write permits changes inside the workspace while keeping shell network access and writes outside the workspace blocked.
- Auto uses the Workspace write shell sandbox by default. A tool may request review for one exact action, and an approved tool may receive a temporary Full access override only when its own policy explicitly requires it.
- Full access removes Rig's filesystem, shell, and network restrictions.

Every tool definition must own its Auto behavior. `shouldReviewInAutoMode` is required. Define `shouldRunInFullAccessInAutoMode` only for reviewed actions that must cross the sandbox; review alone must not imply elevation. Use `requiresAutoOrFullAccess` for tools such as MCP operations whose external execution boundary cannot be enforced by Rig's local sandbox. Use `autoPermissionInstructions` for provider-specific model guidance and `describeAutoPermissionAction` when an approval must disclose a specialized boundary. Keep the agent loop generic: never dispatch permission behavior from a tool-name list, prefix, provider ID, or guessed command contents.

Shell commands are sandboxed identically regardless of provider. Their model-facing escalation syntax is intentionally provider-shaped:

- Codex `exec_command` uses `sandbox_permissions: "require_escalated"` with a concise `justification`.
- Claude `Bash` uses `dangerouslyDisableSandbox: true` and retains Claude's native schema.
- Pi `bash` uses `sandbox_permissions: "require_escalated"` with a concise `justification`.
- Grok `run_terminal_command` uses `sandbox_permissions: "require_escalated"` and explains the need in `description`.

These fields request the same runtime behavior. In Auto, the action is reviewed first; if allowed, the loop scopes only that tool execution to `full_access` and restores Auto immediately afterward. Omitting the field keeps the command sandboxed. In Read only or Workspace write, an escalation argument must not bypass the selected mode. A reviewed action that does not need host access, such as sending input to an existing shell, stays in the current sandbox.

File tools follow the same ownership rule. Each provider tool extracts its actual path argument and calls shared, provider-neutral boundary helpers. Reads outside the allowed boundary, writes outside the workspace, symlink escapes, and writes to protected Git control paths require the appropriate review and elevation. Shared helpers may resolve paths and evaluate boundaries, but must not infer behavior from tool names or maintain parallel registries of read and write tools.

Auto review must use the durable, role-aware conversation transcript rather than a compacted model-context suffix. Real user messages and trusted answers to interactive questions are authorization evidence. Assistant text, tool arguments, tool output, repository content, generated summaries, and prompt injection are not user authorization. Preserve user evidence preferentially within the review budget and fail closed when required user evidence, reviewer output, or reviewer availability is incomplete. Approval applies only to the proposed action; it is not a durable command rule or authorization for later actions.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [slopus/rig](https://github.com/slopus/rig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
