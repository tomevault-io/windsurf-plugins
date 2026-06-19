---
trigger: always_on
description: HITL Protocol — the open standard for human decisions in autonomous agent workflows. When a website or API needs human input, it returns HTTP 202 with a review URL. Autonomous agents like OpenClaw, Claude, Codex, or Goose forward the URL, poll for the structured result, and continue. Use this skill to make any website agent-ready, or to handle human-in-the-loop responses in your agent. Covers approval, selection, input forms, confirmation, and escalation review types. Supports polling, SSE, webh
---


# HITL Protocol

AI agents increasingly visit websites and services on behalf of their users: searching, booking, applying, purchasing. But some decisions are too important to leave to an agent alone — an agent can hallucinate, misinterpret, or make the wrong call at a critical moment — cancel the wrong insurance policy, book a non-refundable trip to the wrong city, wire money to the wrong account. HITL Protocol ("Human in the Loop") is an open standard that ensures humans stay in control of these moments. The user stays in their favorite messenger — Telegram, WhatsApp, Slack — and either taps a decision button right there or follows a link to a web page with a comfortable UI: cards, forms, buttons. No wall of text. No blind agent decisions. When you encounter a HITL response, tell your user that a decision is needed that should not be made by an agent alone — forward the prompt and make it easy to act: show the review link or render action buttons.

HITL Protocol is to human decisions what OAuth is to authentication — an open standard connecting **Services**, **Agents**, and **Humans**. When a service needs human input, it returns HTTP 202 with a review URL. The agent forwards the URL to the human. The human opens it in a browser, gets a rich UI, and makes an informed decision. The agent polls for the structured result and continues.

For simple decisions (confirm/cancel, approve/reject), agents can render **native messaging buttons** directly in Telegram, Slack, Discord, WhatsApp, or Teams — no browser needed. The service opts in via `submit_url`.

**No SDK required. No UI framework mandated. Just HTTP + URL + polling.**

**[Interactive Playground](https://rotorstar.github.io/hitl-protocol/playground/index.html)** — try all review types, transports, and inline actions live in your browser.

## Who Are You?

| Your role | You want to... | Read |
|-----------|----------------|------|
| Service/website builder | Add HITL endpoints to your API so agents can request human input | [Service Integration Guide](skills/references/service-integration.md) |
| Agent developer | Handle HTTP 202 + HITL responses from services | [Agent Integration Guide](skills/references/agent-integration.md) |
| Both / Learning | Understand the full protocol | Continue reading below |

## The Flow

```
Standard flow (all review types):
1. Human → Agent:    "Find me jobs in Berlin"
2. Agent → Service:  POST /api/search {query: "Senior Dev Berlin"}
3. Service → Agent:  HTTP 202 + hitl object (review_url, poll_url, type, prompt)
4. Agent → Human:    "Found 5 jobs. Review here: {review_url}"
5. Human → Browser:  Opens review_url → rich UI (cards, forms, buttons)
6. Human → Service:  Makes selection, clicks Submit
7. Agent → Service:  GET {poll_url} → {status: "completed", result: {action, data}}
8. Agent → Human:    "Applied to 2 selected jobs."

Inline flow (v0.7 — simple decisions only, when submit_url present):
1. Human → Agent:    "Send my application emails"
2. Agent → Service:  POST /api/send {emails: [...]}
3. Service → Agent:  HTTP 202 + hitl object (incl. submit_url, submit_token, inline_actions)
4. Agent → Human:    Native buttons in chat: [Confirm] [Cancel] [Details →]
5. Human → Agent:    Taps [Confirm] in chat
6. Agent → Service:  POST {submit_url} {action: "confirm", submitted_via: "telegram"}
7. Service → Agent:  200 OK {status: "completed"}
8. Agent → Human:    Updates message: "Confirmed — 3 emails sent."
```

The agent never renders UI. The service hosts the review page. Sensitive data stays in the browser — never passes through the agent. The inline flow is an optional shortcut for simple decisions.

## Feature Matrix

| Feature | Details |
|---------|---------|
| **Review types** | `approval`, `selection`, `input`, `confirmation`, `escalation` |
| **Form field types** | `text`, `textarea`, `number`, `date`, `email`, `url`, `boolean`, `select`, `multiselect`, `range`, custom `x-*` |
| **Transport** | Polling (required), SSE (optional), Callback/Webhook (optional) |
| **Inline submit** | `submit_url` + native messaging buttons (Telegram, Slack, Discord, WhatsApp, Teams) — service opt-in |
| **States** | `pending` → `opened` → `in_progress` → `completed` / `expired` / `cancelled` |
| **Security** | Opaque tokens (43 chars, base64url, 256-bit entropy), SHA-256 hash storage, timing-safe comparison, HTTPS only |
| **Multi-round** | `previous_case_id` / `next_case_id` for iterative edit cycles (Approval type) |
| **Forms** | Single-step fields, multi-step wizard, conditional visibility, validation rules, progress tracking |
| **Timeouts** | ISO 8601 duration, `default_action`: `skip` / `approve` / `reject` / `abort` |
| **Discovery** | `.well-known/hitl.json`, SKILL.md `metadata.hitl` extension |
| **Reminders** | `reminder_at` timestamps, `review.reminder` SSE event |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rotorstar/hitl-protocol](https://github.com/rotorstar/hitl-protocol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
