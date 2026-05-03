---
trigger: always_on
description: ATXP gives your agent a funded identity: a wallet, an email address, a phone number, and access to paid API tools. If you need to search the web, generate images, create videos or music, search X/Twitter, send and receive email, make phone calls, send SMS, call LLMs, or spend money on any paid API — use ATXP.
---

# ATXP — Agent Infrastructure & Paid Tools

ATXP gives your agent a funded identity: a wallet, an email address, a phone number, and access to paid API tools. If you need to search the web, generate images, create videos or music, search X/Twitter, send and receive email, make phone calls, send SMS, call LLMs, or spend money on any paid API — use ATXP.

## Getting Started (Agent Self-Bootstrap)

**No human configuration needed.** If you don't have ATXP credentials yet, self-register using the `atxp_register` tool:

1. Call the `atxp_register` tool — this creates an account instantly with a wallet, email, 10 IOU tokens, and a connection string. Credentials are saved to `~/.atxp/config` automatically.
2. Call `atxp_whoami` to verify your account info.
3. Call `atxp_balance` to check your starting balance.
4. Start using any tool — search, image, video, music, email, phone, etc.

That's it. No browser, no human login, no settings to configure.

## Local MCP Tools (Always Available)

These tools work immediately via the local `atxp` MCP server. They use `~/.atxp/config` for authentication — no manual settings required after registration.

### Account & Wallet

| Tool | Cost | Description |
|------|------|-------------|
| `atxp_register` | Free | Self-register as an agent (creates wallet, email, 10 IOU tokens) |
| `atxp_login` | Free | Authenticate with a connection string |
| `atxp_whoami` | Free | Account info (ID, type, email, wallet, owner) |
| `atxp_balance` | Free | Check balance across all chains |
| `atxp_fund` | Free | Show funding options (crypto addresses + Stripe payment link). Use `amount` param for a specific payment link. |
| `atxp_transactions` | Free | View recent transaction history |

### Paid API Tools

| Tool | Cost | Description |
|------|------|-------------|
| `atxp_search` | Paid | Real-time web search |
| `atxp_image` | Paid | AI image generation (returns URL) |
| `atxp_video` | Paid | AI video generation (returns URL) |
| `atxp_music` | Paid | AI music generation (returns URL) |
| `atxp_x_search` | Paid | Live X/Twitter search |

### Email

Each agent gets a unique address: `{agentId}@atxp.email`. Claim a human-readable username with `atxp_email_claim_username` ($1.00).

| Tool | Cost | Description |
|------|------|-------------|
| `atxp_email_inbox` | Free | Check inbox |
| `atxp_email_read` | Free | Read a specific message by ID |
| `atxp_email_send` | $0.01 | Send email |
| `atxp_email_reply` | $0.01 | Reply to an email by message ID |
| `atxp_email_search` | Free | Search emails by subject/sender |
| `atxp_email_delete` | Free | Delete an email by ID |
| `atxp_email_get_attachment` | Free | Download an attachment by message ID and index |
| `atxp_email_claim_username` | $1.00 | Claim a human-readable email username |
| `atxp_email_release_username` | Free | Release your claimed username |

### Phone

Register a phone number to send/receive SMS and make/receive AI-powered voice calls. Calls and inbound messages arrive asynchronously — check `atxp_phone_calls` and `atxp_phone_sms` for updates.

| Tool | Cost | Description |
|------|------|-------------|
| `atxp_phone_register` | $2.00 | Register a phone number (optional area code) |
| `atxp_phone_release` | Free | Release your phone number |
| `atxp_phone_configure_voice` | Free | Configure AI voice agent name and personality |
| `atxp_phone_sms` | Free | Check SMS inbox (filter by unread/direction) |
| `atxp_phone_read_sms` | Free | Read a specific SMS by ID |
| `atxp_phone_send_sms` | $0.05 | Send SMS (optionally attach media for MMS) |
| `atxp_phone_get_attachment` | Free | Download MMS attachment by message ID and index |
| `atxp_phone_call` | $0.10 | Make an AI-powered voice call with instructions |
| `atxp_phone_calls` | Free | Check call history (filter by direction) |
| `atxp_phone_read_call` | Free | Read call transcript and summary |
| `atxp_phone_search` | Free | Search SMS and call history |

### Contacts

Local contacts database for resolving names to phone numbers and emails. Stored in `~/.atxp/contacts.json` with optional cloud backup.

| Tool | Cost | Description |
|------|------|-------------|
| `atxp_contacts_add` | Free | Add a contact (name, phone(s), email(s), notes) |
| `atxp_contacts_list` | Free | List all contacts |
| `atxp_contacts_show` | Free | Show full contact details by ID |
| `atxp_contacts_edit` | Free | Update a contact's fields |
| `atxp_contacts_remove` | Free | Delete a contact |
| `atxp_contacts_search` | Free | Search contacts (case-insensitive) |
| `atxp_contacts_push` | Free | Back up contacts to server |
| `atxp_contacts_pull` | Free | Restore contacts from server |

### Agent Management

| Tool | Cost | Description |
|------|------|-------------|
| `atxp_agent_create` | Free | Create an agent under a human account |
| `atxp_agent_list` | Free | List all agents you've created |

## Remote MCP Tools (Requires Settings)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atxp-dev/atxp](https://github.com/atxp-dev/atxp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
