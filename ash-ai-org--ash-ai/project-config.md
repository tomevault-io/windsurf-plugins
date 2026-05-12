---
trigger: always_on
description: In this codebase, "agent" refers to **Ash's deployed agents** — the AI agents that users deploy and orchestrate via Ash. It does NOT refer to you (the coding agent working on this repo). When this file says "the agent runs inside a sandbox," it means an Ash-deployed agent, not you.
---

# CLAUDE.md

## Terminology

In this codebase, "agent" refers to **Ash's deployed agents** — the AI agents that users deploy and orchestrate via Ash. It does NOT refer to you (the coding agent working on this repo). When this file says "the agent runs inside a sandbox," it means an Ash-deployed agent, not you.

## What Is Ash

Ash is a CLI, SDK, and self-hostable system for deploying and orchestrating hosted AI agents. Developers define agents as folders (CLAUDE.md + config), deploy them to a server, and interact via REST API + SSE streaming. Agents run inside isolated sandboxes.

## Deep-Dive Documentation

For detailed architecture, feature docs, design decisions, and the implementation plan, read `docs/INDEX.md`. It is the table of contents for all project documentation. Start there before exploring the codebase.

## Principles

These govern every decision. When in doubt, re-read these.

### 1. Make It Work, Then Make It Right, Then Make It Fast

Do not optimize what you haven't measured. Do not distribute what runs fine on one machine. Do not abstract what isn't repeated. The correct order is:

1. Get the feature working end-to-end (ugly is fine)
2. Make it correct (handle failures, survive restarts, isolate properly)
3. Measure it (instrument, benchmark, find the actual bottleneck)
4. Only then optimize (and verify the optimization with the same measurements)

If you skip to step 4 you will build something fast and wrong.

### 2. Delete Indirection Until It Hurts

Every layer of indirection — every proxy, every abstraction, every service boundary — has a cost: latency, failure modes, debugging difficulty. That cost must be justified by a concrete, present-tense need, not a hypothetical future one.

If the server and runner are on the same machine, they should be in the same process. If a function is called once, it doesn't need a class. If a pattern isn't repeated three times, it doesn't need a utility. Add the abstraction when the duplication is painful, not when it's theoretically possible.

### 3. Correctness Is Not a Feature, It's a Prerequisite

A system that loses state on restart is not a system — it's a demo. A sandbox that leaks host environment variables is not a sandbox — it's a liability. Correctness means:

- Sessions survive process restarts (persistent state)
- Sandboxes cannot read, write, or signal anything outside their boundaries
- Failure modes are explicit and recoverable, not silent and corrupting
- Every state transition is tested

Ship correct code that does less, not broken code that does more.

### 4. Measure Before and After

Never commit an optimization without before/after numbers. Never claim a feature works without a test that exercises it. The questions that matter:

- What is the p99 latency from message-in to first-token-out?
- What is Ash's overhead on top of the SDK itself?
- How many concurrent sessions before degradation?
- Where is time actually spent?

Numbers in `docs/` or it didn't happen.

### 5. The Test Is the Spec

If the behavior isn't tested, it isn't guaranteed. Tests encode what the system promises. When requirements change, change the test first, then change the code. Specifically:

- Test boundaries (protocol serialization, API contracts, state transitions)
- Test failure modes (crash mid-stream, disconnect, timeout, corrupt input)
- Test invariants (sandbox env never contains host secrets, ended session rejects messages)
- Don't test glue (trivial wrappers, type re-exports, config loading)

### 6. Security Is a Constraint, Not a Phase

Every sandbox process runs with a restricted environment from day one. Not "we'll add isolation later." The allowlist approach: sandbox env contains only explicitly permitted variables. Everything else is denied by default. The threat model is simple: the agent inside the sandbox is untrusted code running arbitrary shell commands. Design accordingly.

### 7. Ship Incrementally

Every change should be independently shippable and independently revertable. Don't batch six things into one PR. Don't leave the system in a half-migrated state. Each step in the plan stands alone — it improves something, it has tests, it doesn't require the next step to be useful.

### 8. Ash Is a Thin Wrapper — Use the SDK's Types

Ash orchestrates the Claude Code SDK (`@anthropic-ai/claude-code`). It does not reinvent it. The SDK already defines good types (`Message`, `AssistantMessage`, `ResultMessage`, `ToolUseBlock`, etc.), a streaming interface (`query()` / `Session`), and options (`Options`, `PermissionMode`, etc.). Ash should pass these through, not translate them.

Concretely:

- **Bridge events should be SDK messages.** The bridge process yields `Message` objects from the SDK. Send them over the Unix socket as-is (newline-delimited JSON). Don't invent `BridgeAssistantMessageEvent` / `BridgeToolUseEvent` when the SDK already has `AssistantMessage` / `ToolUseBlock`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ash-ai-org/ash-ai](https://github.com/ash-ai-org/ash-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
