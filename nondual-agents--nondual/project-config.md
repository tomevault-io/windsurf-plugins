---
trigger: always_on
description: The address book for AI agents.
---

# Nondual — Agent Onboarding

The address book for AI agents.

This file is read automatically by Cursor, Claude Code, and other AI agents.
It tells you what Nondual does, how to install it, and how to use every tool.

For the full REST reference: https://nondual.cloud/docs
For the live playground: https://nondual.cloud

---

## What Nondual does

One API key gives every agent you run access to a shared workspace. Any agent can:

- Look up a person by email, LinkedIn URL, phone, or handle — get their full profile,
  interaction history, open follow-ups, and a recommended next action.
- Record any outreach (email, call, meeting, message) and attach a follow-up in one call.
- List all open follow-ups due today, by owner, or by company domain.
- Pull every contact and interaction at a company by domain.
- Bulk-import contacts, interactions, and follow-ups from a CSV or CRM export.

All agents on the same key share the same memory. Ten agents work like one team.

---

## Setup

### Step 1 — Get a key

**You are an agent. Use this path — the key comes back in the API response.**

```bash
curl -s -X POST https://api.nondual.cloud/v1/keys \
  -H "Content-Type: application/json" \
  -d '{"email": "HUMAN_EMAIL", "requested_by": "agent"}'
```

Response:

```json
{
  "api_key": "nd_xxxxxxxxxxxxxxxxxxxx",
  "status": "unclaimed",
  "message": "Key issued. A claim code was sent to HUMAN_EMAIL."
}
```

Store the key. Set it as `NONDUAL_API_KEY` in your environment or project secrets.

The human claims it at https://nondual.cloud/claim — this lifts workspace restrictions.
Until claimed, the key works for contacts you create and for the signup email contact.

### Step 2 — Choose your interface

**MCP (Cursor, Claude Code, Windsurf, any MCP host):**

```json
{
  "mcpServers": {
    "nondual": {
      "url": "https://mcp.nondual.cloud/mcp",
      "headers": { "Authorization": "Bearer YOUR_API_KEY" }
    }
  }
}
```

**CLI:**

```bash
npx nondual init --agent --email HUMAN_EMAIL
npx nondual get-contact-info EMAIL_OR_URL
npx nondual record-contact-interaction EMAIL --channel email --summary "what happened"
npx nondual list-open-followups
npx nondual get-company-activity DOMAIN
```

**TypeScript SDK:**

```bash
npm install nondual
```

```typescript
import { Nondual } from 'nondual';
const nd = new Nondual(); // reads NONDUAL_API_KEY from env
```

**REST — all endpoints at `https://api.nondual.cloud/v1/`:**

```bash
curl -X POST https://api.nondual.cloud/v1/get-contact-info \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"contact": "email@example.com"}'
```

### Step 3 — Verify

```bash
npx nondual get-contact-info dario@anthropic.com
npx nondual record-contact-interaction dario@anthropic.com \\
  --summary "agent verified nondual MCP setup"
```

You should get a full profile for Dario and a 201 confirming the interaction was recorded.
Call `get-contact-info` again and confirm the interaction appears in the timeline.
That is the read-write loop. Nondual is set up.

---

## The four tools

### get-contact-info

Look up a person. Returns full profile, relationship summary, recent interactions, open follow-ups,
and recommended next action.

```json
POST /v1/get-contact-info
{
  "contact": "dario@anthropic.com",
  "enrich": true
}
```

**Always call this before any outreach.** It tells you:
- Whether `do_not_disturb` is set (if true, do not reach out)
- What your agents have already said to this person
- What the recommended next action is

**`also` — multi-identifier merge:**

When you have both an email and a LinkedIn URL for the same person, pass both:

```json
{
  "contact": "kai@company.com",
  "also": ["https://linkedin.com/in/kuhlig"]
}
```

The system treats this as ground truth. Both identifiers are merged into one contact,
enrichment runs on every identifier, and all alias keys are stored. This is the correct
pattern for LinkedIn connections exports where the CSV email may not match enrichment data.

**Fields:**
- `contact` (required) — email, LinkedIn URL (`linkedin.com/in/slug`), phone (E.164), or @handle
- `enrich` (optional, default `true`) — set `false` for workspace-only lookup, no vendor calls
- `also` (optional) — array of additional identifiers for the same person

### record-contact-interaction

Record any outreach. Unknown contacts are created on the fly — no prior lookup required.

```json
POST /v1/record-contact-interaction
{
  "contact": "dario@anthropic.com",
  "channel": "email",
  "direction": "outbound",
  "summary": "Sent intro about AI safety partnership",
  "details": "Hi Dario, I wanted to reach out about a potential AI safety research collaboration. Happy to share more.",
  "followup_action": "Send proposal deck",
  "followup_due": "2026-08-01"
}
```

**Required:** `contact`, `summary`

**Communication (optional):** `channel`, `direction`, `occurred_at`, `details` (full email body/transcript), `participants`

**Attribution (optional):** `recorded_by: { agent, user }`, `source: { platform, external_id }`

**Follow-ups (optional):** `followup_action`, `followup_due`, `followup_agent`, `complete_followups` (array of IDs or `"all"`)

**Contact state (optional):** `do_not_disturb: true` sets the flag; `false` clears it

### list-open-followups


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nondual-agents/nondual](https://github.com/nondual-agents/nondual) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
