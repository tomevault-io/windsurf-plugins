---
trigger: always_on
description: Use when building notifications with Courier across email, SMS, push, in-app inbox, Slack, Teams, or WhatsApp. Covers transactional messages (password reset, OTP, orders, billing), growth notifications (onboarding, engagement, referral), multi-channel routing, preferences and topics, reliability and webhooks, journeys (multi-step notification sequences via API), template CRUD and Elemental content, routing strategies, provider configuration, the Courier CLI and MCP server, and migrations from Kn
---


# Courier Notification Skills

Guidance for building deliverable and engaging notifications across all channels.

## How to Use This Skill

1. **Identify the task** — What channel, notification type, or cross-cutting concern is the user working on?
2. **Read only what's needed** — Use the routing tables below to find the 1-2 files relevant to the task. Do NOT read all files.
3. **Check for live docs** — For current API signatures and SDK methods, fetch `https://www.courier.com/docs/llms.txt`
4. **Synthesize before coding** — Plan the complete implementation (channels, routing, error handling) before writing code.
5. **Apply the rules** — Each resource file starts with a "Quick Reference" section containing hard rules. Treat these as constraints, not suggestions.
6. **Check universal rules** — Before generating any notification code, verify it doesn't violate the Universal Rules below.

## Handling Vague Requests

If the user's request doesn't clearly map to a specific channel, notification type, or guide, **ask clarifying questions before reading any resource files**. Don't guess — a wrong routing wastes time and produces irrelevant code.

**Ask these questions as needed:**

1. **What channel?** — "Which channel are you sending through: email, SMS, push, in-app, Slack, Teams, or WhatsApp?"
2. **What type?** — "Is this a transactional notification (triggered by a user action, like a password reset or order confirmation) or a marketing/growth notification (sent proactively, like a feature announcement)?"
3. **New or existing?** — "Are you starting from scratch, or do you have existing Courier code? If existing, what SDK packages do you have installed?"
4. **What language?** — "Are you using TypeScript/Node.js, Python, or another language?"

You don't need to ask all four — just the ones needed to route to the right 1-2 files. If the request is clearly about a specific topic (e.g., "help me with SMS"), skip the questions and go directly to the relevant resource.

**Routing consequences of question 3 ("new or existing"):**

| Answer | Skip | Load |
|--------|------|------|
| New to Courier / no existing code | (nothing) | [quickstart.md](./resources/guides/quickstart.md) + the relevant channel or type file |
| Existing — has `@trycourier/courier` or `trycourier` installed | `quickstart.md` install + env-setup sections | Jump directly to channel or type file; assume `client` is constructed. Offer `courier messages list` as a one-line health check if useful. |
| Existing — Inbox v7 (`@trycourier/react-*`) | v8 guidance | See "Courier Inbox Version Detection" block below, then [inbox-v7-legacy.md](./resources/channels/inbox-v7-legacy.md) |

## Canonical SDK Shape

Before you write or evaluate any Courier code, ground it in this shape. If anything in a file below appears to contradict it, trust this block and fetch live docs to resolve — do **not** paste the contradicting snippet.

**Node.js (`@trycourier/courier`, Stainless-generated):**

```typescript
import Courier from "@trycourier/courier";

// Reads process.env.COURIER_API_KEY by default
const client = new Courier();

await client.send.message({
  message: {
    to: { user_id: "user-123" },           // or { email }, { phone_number }, { list_id }, { tenant_id }, etc.
    template: "nt_01kmrbq6ypf25tsge12qek41r0", // OR content: { title, body } / { version, elements }
    data: { /* merge variables */ },
  },
}, {
  // Pass the Idempotency-Key via headers. Always set it explicitly here —
  // that is the one path guaranteed to be sent to the API across SDK
  // versions. Verify against your installed SDK version before relying on
  // any other `idempotencyKey` request option.
  headers: { "Idempotency-Key": "order-confirmation-12345" },
});
```

**Python (`trycourier`, Stainless-generated):**

```python
from courier import Courier

# Reads COURIER_API_KEY from env by default
client = Courier()

client.send.message(
    message={
        "to": {"user_id": "user-123"},
        "template": "nt_01kmrbq6ypf25tsge12qek41r0",
        "data": {},
    },
    # Pass the Idempotency-Key via extra_headers. Python does not accept
    # idempotency_key= as a keyword argument — the header is the only way.
    extra_headers={"Idempotency-Key": "order-confirmation-12345"},
)
```

**Method naming quick lookup (generated SDKs — both SDKs follow the same structure, Node = camelCase, Python = snake_case):**

| Operation | Node | Python |
|-----------|------|--------|
| Send a message | `client.send.message({ message })` | `client.send.message(message=...)` |
| Create a template | `client.notifications.create({ notification, state })` → returns `{ id, name, content, … }` at top level | `client.notifications.create(notification=..., state=...)` → `response.id` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trycourier/courier-skills](https://github.com/trycourier/courier-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
