---
trigger: always_on
description: Comprehensive Epsimo AI platform SDK and CLI for managing agents, projects, threads, Virtual Database, and frontend design. Build AI apps with persistent state, streaming conversations, and React UI kit.
---


# Epsimo Agent Framework

> [!NOTE]
> This is a **Beta Release** (v0.3.0). Features and APIs may evolve based on feedback.

The Epsimo Agent Framework allows you to build sophisticated AI-powered applications with agents, persistent threads, and a "Virtual Database" state layer. It provides a unified **CLI** (with TUI dashboard and smart commands), a **Python SDK**, and a **React UI Kit**.

**Base URL:** `https://api.epsimoagents.com`  
**Frontend URL:** `https://app.epsimoagents.com`

---

## 🚀 Quick Start: Create an MVP in 3 Commands

The fastest way to build an Epsimo app is using the project generator:

```bash
# 1. Authenticate
epsimo auth

# 2. Create a new Next.js project
epsimo create "My AI App"

# 3. Initialize and Deploy
cd my-ai-app
epsimo init
epsimo deploy
```

Your AI app is now live with persistent conversations, Virtual Database state, and a pre-built chat UI.

---

## 📦 Installation

### One-Line Install (Recommended)

```bash
# macOS / Linux
curl -fsSL https://raw.githubusercontent.com/thierryteisseire/epsimo-agent/main/install.sh | bash

# Windows (PowerShell)
irm https://raw.githubusercontent.com/thierryteisseire/epsimo-agent/main/install.ps1 | iex
```

### Via npm

```bash
npm install -g epsimo-agent
```

### From Source (Python)

```bash
pip install -r requirements.txt
pip install -e .
```

**Dependencies:**
- `requests>=2.28.0`
- `pyyaml>=6.0`
- Python 3.8+

---

## 🔐 Authentication

### Login Flow

```bash
# Interactive login
epsimo auth login

# Login with environment variables
export EPSIMO_EMAIL=your@email.com
export EPSIMO_PASSWORD=your-password
epsimo auth
```

### Programmatic Authentication

```python
from epsimo.auth import perform_login, get_token

# Login and get token
token = perform_login("your@email.com", "password")

# Get cached token (auto-refreshes if expired)
token = get_token()
```

### Token Management

Tokens are stored in `~/code/epsimo-frontend/.epsimo_token` and expire after **1 hour**.

**Token Storage Format:**
```json
{
  "access_token": "eyJhbG...",
  "token": "eyJhbG...",
  "jwt_token": "eyJhbG...",
  "expires_in": 3600
}
```

**Best Practices:**
- ✅ Use environment variables (`EPSIMO_API_KEY`) for production
- ✅ Never commit `.epsimo_token` to version control
- ✅ Implement token refresh on 401 errors
- ✅ Use project-specific tokens for multi-tenant apps

---

## 🛠️ Unified CLI (`epsimo`)

The `epsimo` CLI is the main tool for managing your agents and data.

### Authentication Commands

| Command | Description |
|---------|-------------|
| `epsimo auth` | Interactive login with email/password |
| `epsimo whoami` | Display current user info and thread usage |

**Example:**
```bash
$ epsimo whoami
👤 Fetching user info...
Logged in as: user@example.com
Threads Used: 45/100
```

### Project Management

| Command | Description |
|---------|-------------|
| `epsimo projects` | List all projects |
| `epsimo projects --json` | List projects in JSON format |
| `epsimo create <name>` | Scaffold a full Next.js application with Epsimo |
| `epsimo init` | Link a local directory to an Epsimo project |
| `epsimo deploy` | Sync your `epsimo.yaml` configuration to the platform |

**Example:**
```bash
$ epsimo projects
📁 Fetching projects...
ID                                      | Name
-----------------------------------------------------------------
proj_abc123                             | Customer Support Bot
proj_xyz789                             | Research Assistant
```

### Virtual Database Commands

Threads can serve as a persistent structured storage layer.

| Command | Description |
|---------|-------------|
| `epsimo db query --project-id <P_ID> --thread-id <T_ID>` | View all structured thread state |
| `epsimo db get --project-id <P_ID> --thread-id <T_ID> --key <K>` | Get specific key from thread state |
| `epsimo db set --project-id <P_ID> --thread-id <T_ID> --key <K> --value <V>` | Seed state for testing |

**Example:**
```bash
$ epsimo db query --project-id proj_abc --thread-id thread_123
{
  "user_preferences": {
    "theme": "dark",
    "language": "en"
  },
  "status": "active"
}

$ epsimo db set --project-id proj_abc --thread-id thread_123 \
  --key "status" --value '"completed"'
```

### Credits & Billing

| Command | Description |
|---------|-------------|
| `epsimo credits balance` | Check current thread balance and subscription tier |
| `epsimo credits buy --quantity <N>` | Generate a Stripe checkout URL for purchasing threads |

**Example:**
```bash
$ epsimo credits balance
💳 Checking balance...

=== Thread Balance ===
Threads Used:      45
Total Allowance:   100
Threads Remaining: 55
======================

$ epsimo credits buy --quantity 1000
🛒 Preparing purchase of 1000 threads...
ℹ️  Estimated cost: 80.0 EUR

✅ Checkout session created successfully!
Please visit this URL to complete your purchase:

https://checkout.stripe.com/session_...
```

**Pricing:**
- < 500 threads: €0.10/thread
- 500-999 threads: €0.09/thread
- 1000+ threads: €0.08/thread

### Resource Listing

| Command | Description |
|---------|-------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thierryteisseire/Epsimo-agent](https://github.com/thierryteisseire/Epsimo-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
