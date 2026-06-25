---
trigger: always_on
description: A local trace viewer for Claude Code conversations. Reads session transcripts directly from disk and renders them as a span-tree waterfall. A React + Vite app with a small middleware plugin — no separate server process, no hooks, no OTEL SDK.
---

# agent-profiler

A local trace viewer for Claude Code conversations. Reads session transcripts directly from disk and renders them as a span-tree waterfall. A React + Vite app with a small middleware plugin — no separate server process, no hooks, no OTEL SDK.

## Communication style

Speak simply and explain things clearly. Avoid jargon — when a domain term is genuinely necessary (e.g. `requestId`, `tool_use`), define it the first time it appears in a response. Prefer plain-English descriptions over acronyms and library-internal naming. If a concept can be explained with a short sentence instead of a technical label, use the sentence.

## North Star

**Claude Code's own per-session JSONL transcripts (`~/.claude/projects/<project-slug>/<sessionId>.jsonl`) are the source of truth.** The viewer is a pure `(transcript → SpanNode tree → JSON)` pipeline. If the UI and the raw transcript disagree about what happened, the transcript is right.

## Core vocabulary — inference vs. tool call

These two are not the same thing. Confusing them is the largest recurring source of bugs in this transformer.

**Inference.** A single request to the remote Anthropic API. Something that costs money, takes wall-clock time, and ends with the model returning a response. Has a `requestId` — one per API round-trip. Whatever content the model returned (thinking, text, tool_use, or any combination) belongs to that one inference and shares its `requestId`. **The state layer mirrors API round-trips 1:1: one `inference` span per distinct `requestId` within a slice**, regardless of what content kinds the response contained. A response with `[thinking, text, tool_use]` is one inference, not three. *How* that inference is rendered (waterfall bar, chat bubble, both, neither) is a UI decision driven by `has_*` flags and the events list — the state layer is content-kind-agnostic.

The canonical predicate for "an assistant record that counts" is `isCanonicalAssistantRecord` (`type === 'assistant'`, `!isApiErrorMessage`, has `message.usage`, has non-empty `requestId`). It gates both the inference emitter *and* token dedupe (`dedupeUsagesByRequestId` in `lib/traces/traces.js`) — `inferences.length` equals `agent_trace.turn.request_count` by construction. Drift here recreates the count discrepancy on any turn with API errors.

**Tool call (`tool_use` content block).** Local execution on the user's machine — Bash, Read, Edit, Glob, Grep, etc. **Tool calls are not inferences.** They are produced *inside* an assistant response as a content block, but executing them is local — the agent harness runs the command, captures the result, and feeds it back into the *next* inference as added context. A `tool_use` block carries a `requestId` because the inference that emitted it had one; that does not make the local execution itself an API round-trip. A single inference can emit zero, one, or many `tool_use` blocks; each one becomes its own tool span (duration `tool_result.ts − tool_use.ts`), parented under the inference that emitted it.

**Tool result.** A `user` JSONL record whose `message.content` contains `tool_result` blocks is the *output* of one or more tool executions. It has no `requestId` of its own (no API call happened). It is paired to its `tool_use` via `tool_use_id`. Tool results are not inferences either; they are the *input* that the next inference reads.

**Why this matters.** Conflating tool calls with inferences inflates inference counts and misattributes cost. The user mental model — "how many times did we call the model?" — is exactly `requestId`-count. Tool calls happen between inferences; they shape the prompt for the *next* inference but are never themselves inferences.

**In the transcript.**

- An `assistant` JSONL record carries `requestId`. The record's `message.content` is an array of content blocks (`text`, `thinking`, `tool_use`, …). Claude Code flushes each content block as its own JSONL row, so one API response ≈ N consecutive assistant records sharing one `requestId`.
- A `user` JSONL record with `tool_result` content is the local execution output. No `requestId`. Pairs to its `tool_use` via `tool_use_id`.

**Inference span shape.** Rows of the same `requestId` are aggregated into one span: `usage` is taken once (identical across rows), `stop_reason` from the row that carries it, and the union of content-block types determines the `agent_trace.inference.has_*` booleans (`has_thinking`, `has_text`, `has_tool_use`). Content payloads attach as OTel-semconv events on the inference span — `gen_ai.assistant.reasoning` and `gen_ai.assistant.message` — emitted iff non-empty after truncation.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DevonPeroutky/agent-profiler](https://github.com/DevonPeroutky/agent-profiler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
