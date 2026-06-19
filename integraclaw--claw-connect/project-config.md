---
trigger: always_on
description: Connect to Google Workspace, GitHub, HubSpot, Slack, Notion, Trello, and 20+ more services with managed OAuth, and process documents (extract text, extract structured data, generate PDF/Excel/DOCX). Use this skill when users want to interact with any supported service, extract text or structured data from documents, or generate documents. Security: The INTEGRACLAW_API_KEY authenticates with Integraclaw but grants NO access to third-party services by itself. Each service requires explicit OAuth a
---


# Claw Connect

Semantic actions for Google Workspace, GitHub, HubSpot, Slack, Notion, Trello, and more — provided by [Integraclaw](https://integraclaw.dev). Integraclaw lets you call service actions through a single unified API.

IMPORTANT: OAuth connections are managed by users through the Integraclaw dashboard. The agent's role is to **list available connections** and **call actions** using the references for each service. The agent never manages OAuth flows, tokens, or connection setup. **Processing tools** (extract_text, extract_data, generate_pdf, generate_excel, generate_docx) do not require OAuth connections — they work with just the API key.

## Setup

Follow these steps to configure Integraclaw for the first time. If the user already has `INTEGRACLAW_API_KEY` set, skip to [Quick Start](#quick-start).

### Step 1 — Create an account

Go to [https://integraclaw.dev/register](https://integraclaw.dev/register) and create a free account. If the user already has an account, sign in at [https://integraclaw.dev/login](https://integraclaw.dev/login).

### Step 2 — Create an API key

1. Open the dashboard: [https://integraclaw.dev/dashboard/api-keys](https://integraclaw.dev/dashboard/api-keys)
2. Click **"Create New Key"**
3. Give it a name (e.g. "DevClaw", "OpenClaw", "My Agent")
4. **Copy the key immediately** — it starts with `ic_` and is only shown once

### Step 3 — Set environment variable

```bash
export INTEGRACLAW_API_KEY="ic_YOUR_KEY_HERE"
```

For persistent configuration, add this to your shell profile (`~/.bashrc`, `~/.zshrc`) or your agent's `.env` file.

### Step 4 — Connect a Google service

1. Open [https://integraclaw.dev/dashboard/connections](https://integraclaw.dev/dashboard/connections)
2. Find the service you want (Gmail, Calendar, Drive, Sheets, etc.)
3. Click **"+ Connect"** on the service card
4. A popup opens with Google's OAuth consent screen
5. Sign in with your Google account and grant the requested permissions
6. The popup closes automatically — the service is now connected

Repeat for each Google service you want to use.

### Step 5 — Verify

```bash
curl -s "https://integraclaw.dev/api/v1/connections" \
  -H "Authorization: Bearer $INTEGRACLAW_API_KEY"
```

You should see your connected services:

```json
[
  {
    "id": "...",
    "provider": "google",
    "service": "gmail",
    "email": "user@gmail.com",
    "status": "connected"
  }
]
```

If the response is an empty array `[]`, no services are connected yet — go back to Step 4.

If you get a 401 error, check that `INTEGRACLAW_API_KEY` is set correctly.

## Quick Start

```bash
# List available connections
curl -s "https://integraclaw.dev/api/v1/connections" \
  -H "Authorization: Bearer $INTEGRACLAW_API_KEY"

# Call an action (see references for params)
curl -s -X POST "https://integraclaw.dev/api/v1/action" \
  -H "Authorization: Bearer $INTEGRACLAW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"provider":"google","service":"gmail","action":"send_email","params":{"to":"user@example.com","subject":"Hello!","body":"Sent via Integraclaw"}}'
```

## Base URL

```
https://integraclaw.dev/api/v1
```

## Authentication

All requests require the Integraclaw API key in the Authorization header:

```
Authorization: Bearer $INTEGRACLAW_API_KEY
```

## How It Works

1. **User connects services** through the Integraclaw dashboard (OAuth or API key)
2. **Agent lists connections** via `GET /api/v1/connections` to see what's available
3. **Agent calls actions** via `POST /api/v1/action` — Integraclaw handles token management automatically
4. **Agent consults references** to know which actions exist and what params they accept

The agent never manages OAuth flows, tokens, or connection setup. That's all handled by Integraclaw.

## List Connections

Check which services the user has connected:

```bash
curl -s "https://integraclaw.dev/api/v1/connections" \
  -H "Authorization: Bearer $INTEGRACLAW_API_KEY"
```

**Response:**
```json
[
  {
    "id": "21fd90f9-...",
    "provider": "google",
    "service": "gmail",
    "email": "john@company.com",
    "status": "connected"
  },
  {
    "id": "a7a7b0c5-...",
    "provider": "google",
    "service": "gmail",
    "email": "john.personal@gmail.com",
    "status": "connected"
  }
]
```

Each connection has `provider`, `service`, `status`, and `email` (the account used during OAuth). Only use connections with `status: "connected"`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Integraclaw/claw-connect](https://github.com/Integraclaw/claw-connect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
