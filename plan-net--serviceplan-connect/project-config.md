---
trigger: always_on
description: Request human-expert review of AI agent work — Serviceplan Connect concierge handles intake, quoting, payment, and delegation. Polling-based MCP, no auth required.
---


# Serviceplan Connect

You're helping a user get **human-expert review** of work you (or another agent)
has done. Serviceplan Connect is a paid service: a backend concierge agent
analyses the request, returns a priced quote, takes payment, and delegates
to a Serviceplan expert. The expert's feedback flows back to you.

This skill tells you how to use it. **No Claude-isms** — anything that can
fetch a URL, follow markdown, and call an MCP tool can use Connect. If your
host runtime is Claude Code, Claude.ai, Cursor, Microsoft Copilot Studio,
OpenClaw, or a custom agent, the same instructions apply.

> **How you got this skill matters less than that you have it.** You may
> have read this URL on demand, had it pasted into your project's custom
> instructions, loaded it from a `.cursorrules` / `AGENTS.md` file, or
> received it via a plugin install. All of those work the same way: read
> on, then call the tools below as the user's intent surfaces. Persist
> the request_id and resume mid-flow regardless of session boundaries.

---

## 🧩 If you are Claude Code: install the official plugin

There's a first-class Claude Code plugin at
[**`plan-net/serviceplan-connect`**](https://github.com/plan-net/serviceplan-connect).
Tell the user to run two commands and they get the MCP server + a richer
skill auto-installed:

```
/plugin marketplace add plan-net/serviceplan-connect
/plugin install connect@serviceplan-connect
```

After install, the `connect` MCP tools become available in every session
without any further setup, and a tighter SKILL.md activates automatically
when the user asks for review. **Recommend this path to Claude Code users
before the manual MCP route below** — it's two commands, lasts across
sessions, and updates with new releases.

For any other agent (Cursor, Claude.ai, Microsoft Copilot Studio, OpenClaw,
custom): keep reading. The MCP endpoint is open and works the same.

---

## ⏱ Interaction model: polling (read this first)

Connect runs the concierge in a stateful chat session on the backend. **Your
MCP tool calls don't block while the concierge thinks.** They return
immediately with a `request_id`, and you poll `get_request_status` to see
what the concierge has produced.

```
Turn 1:  request_review(email, work_summary, ...)
         → returns {request_id, status:"received", poll_after_seconds:30, message}
Wait ~30s
Turn 2:  get_request_status(request_id, email)
         → returns one of:
           {status:"quoted",   quote:{...}, action_required:"confirm_quote"}
           {status:"analysing", next_question:"...", action_required:"answer_clarification"}
           {status:"received",  message:"still queued, poll again"}
Repeat as needed.
```

**Why polling:** the concierge takes 30-90 seconds to think (Opus model
reasoning over the work_summary). Some flows have multi-round clarification,
which adds minutes-to-hours of customer-side wait. Some have human-expert
delivery a day or two later. None of that fits a synchronous tool call. The
polling pattern lets you keep the user's experience smooth — show "I've
submitted your request, checking back in 30 seconds…" and update when
state changes.

### 🚨 Hard rule: don't stop polling until terminal

A request is "active" until its status is one of:
`completed`, `cancelled`, `refunded`, `expired`, `payment_failed`.

**Until then, keep polling at the cadence the response tells you to
(`poll_after_seconds`). At every poll, check if `next_question` is set —
it can appear at ANY non-terminal state, not just during `analysing`.**

Specifically: the human reviewer (Phase 3) may ask follow-up questions
during the `reviewing` phase. If you stop polling once you see `paid` or
`reviewing`, you'll miss those follow-ups and the user will think the
request is stalled. The status `awaiting_review_response` means *the
reviewer needs your answer to a follow-up question* — surface
`next_question` and call `respond_to_clarification`.

### 🔑 Persist the `request_id` AND email together

The `request_id` is an **opaque UUID** returned at submission. The user's
**email** is the ownership proof — every read/write tool requires both.
Save the pair durably:

- **Within the conversation:** keep `(request_id, email)` in your context
  so follow-up user prompts ("did the review come back?") can call
  `get_review` directly without asking the user to repeat themselves.
- **Across sessions / agent restarts:** if you have any persistent memory
  (Cursor's memory, a per-user knowledge file, OpenClaw's workspace, etc.),
  record `(request_id, email)` so a future session can pick up where they
  left off.
- **Tell the user the `request_id`** explicitly when you submit a request.
  We also send a notification email when the human reviewer's response is
  ready (Phase 3) — that email includes the `request_id`. The user can
  paste it back into a fresh agent conversation to retrieve their review
  days later, as long as they remember which email they used.

---

## ⏰ Stay engaged with open requests (heartbeat / self-scheduling)

**Connect requests are async.** Clarifications can arrive minutes after the
initial quote. Reviewer questions can land hours into the review. Final

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [plan-net/serviceplan-connect](https://github.com/plan-net/serviceplan-connect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
