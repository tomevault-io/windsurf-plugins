---
trigger: always_on
description: agent-sh is designed to be backend-agnostic. The agent that drives a query — assembling context, calling an LLM, executing tools in a loop — is a replaceable component. Any extension can register a backend via `agent:register-backend` and become the default via the `defaultBackend` setting or the `/backend` slash command. Bridge backends like `claude-code` and `pi` plug external CLI agents into the same shell and TUI surface.
---

# The Built-in Agent: ash

agent-sh is designed to be backend-agnostic. The agent that drives a query — assembling context, calling an LLM, executing tools in a loop — is a replaceable component. Any extension can register a backend via `agent:register-backend` and become the default via the `defaultBackend` setting or the `/backend` slash command. Bridge backends like `claude-code` and `pi` plug external CLI agents into the same shell and TUI surface.

This document describes **ash**, the built-in backend that ships with agent-sh. The agent host (`src/agent/index.ts`) is activated unconditionally via `activateAgent(ctx)` — it attaches the `ctx.agent` surface, registers core tools, and emits `agent:register-backend` to register `ash` with the core's backend registry. ash only *activates* (constructs its `AgentLoop` and starts handling queries) when an LLM provider has both an apiKey and a model resolved, and `activateBackend("ash")` runs. It resolves providers from registered catalogs + settings overlay, configures an `LlmClient`, and calls any OpenAI-compatible API directly. It manages conversation state and executes tools in a loop until the LLM is done.

If you're looking to write your own backend instead of reading how ash works internally, see [Extensions: Custom Agent Backends](extensions.md#custom-agent-backends).

## The Query Flow

Here's what happens when you submit a query:

```
User types "> fix the failing test"
  │
  ├─ 1. Context assembly — gather recent shell commands, output, cwd
  ├─ 2. System prompt (cached per cwd) + dynamic context (rebuilt every LLM call)
  ├─ 3. LLM call — stream response from the API
  ├─ 4. Tool loop — if LLM requested tool calls:
  │     ├─ Execute each tool (with permission check if needed)
  │     ├─ Add results to conversation
  │     └─ Go back to step 3 (LLM sees tool results, decides next action)
  └─ 5. Done — no more tool calls, emit response
```

The key insight: **the agent is a loop, not a single call**. The LLM calls tools, sees results, calls more tools, until it has enough information to respond. A single query might trigger dozens of LLM calls and tool executions.

## Context Assembly

Every query draws on two distinct streams of context:

- **Shell context** — the user's terminal activity (commands + outputs) plus the live cwd. This is what lets ash understand "fix this" after you ran a failing command, and what keeps it anchored in the right working directory across compactions. The current cwd is wrapped as `<cwd>` (always) and new shell activity since the last turn as `<shell_events>` (when there is any), both nested inside the per-query `<query_context>` envelope and prepended to your user message.
- **Conversation state** — the OpenAI chat messages array (`user`/`assistant`/`tool` messages). This is the LLM's memory of what it already said and did within this session.

The two streams don't overlap: agent tool outputs live only in the conversation, and shell context tracks only user-initiated activity. When either stream grows large, ash has escape hatches rather than silent truncation:

- **Long shell outputs** are spilled to tempfiles (`<tmpdir>/agent-sh-<pid>/<id>.out`) at capture time. The LLM sees a head+tail stub with the path and recovers the full output via plain `read_file`.
- **Older conversation turns** are compacted by the built-in `rolling-history` extension: each is nucleated into a one-line summary in a persistent store (`~/.agent-sh/rolling-history/history.jsonl`), with the full message kept in an ephemeral per-session cache. The `conversation_recall` tool browses, searches, and expands those entries.

Compaction is pluggable: the `conversation:compact` handler is advisable, so extensions can install richer strategies without changing the recall surface. See [Context Management](context-management.md) for the full design.

## System Prompt

The system prompt is assembled once per `cwd` and cached (invalidated when the working directory changes), so the prefix is stable for provider-side prompt caching. It includes:

1. **Identity** — "You are ash, an AI coding assistant running inside agent-sh..." (advisable via `system-prompt:identity`)
2. **Frontend surface** — the active frontend's self-description, placed right after the identity (advisable via `system-prompt:frontend`; omitted when none)
3. **Static guide** — agent-sh's own code map (paths to `docs/`, `src/`, `examples/extensions/`), generic tool guidance, and the `<query_context>`/`<dynamic_context>` envelope contract
4. **Global memory** — `~/.agent-sh/AGENTS.md`, if present
5. **Global skills** — discovered global skills (cwd-stable)
6. **Project conventions + skills** — `CLAUDE.md`/`AGENT.md` walked from cwd to root, plus discovered project skills (cwd-stable; see next section)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [guanyilun/agent-sh](https://github.com/guanyilun/agent-sh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
