---
trigger: always_on
description: > **Purpose:** This file provides comprehensive guidance to Claude Code when working with this codebase. It serves as the authoritative reference for system architecture, AI behavior rules, and development workflows.
---

# CLAUDE.md

> **Purpose:** This file provides comprehensive guidance to Claude Code when working with this codebase. It serves as the authoritative reference for system architecture, AI behavior rules, and development workflows.

---

## ⚠️ CRITICAL: Testing Account Information

**DO NOT FORGET THIS. EVER.**

| Role | Account | Notes |
|------|---------|-------|
| **Sending Account (Outbox)** | `baylor.freelance@outlook.com` | This is the ONLY account used for sending emails during development/testing |
| **Test Broker 1** | `bp21harrison@gmail.com` | Simulates broker replies |
| **Test Broker 2** | `baylor@manifoldengineering.ai` | Simulates broker replies |

**The product is being built FOR Jill Ames (jill@mohrpartners.com), but ALL testing and development uses baylor.freelance@outlook.com as the sending account.**

**NEVER reference jill@mohrpartners.com when discussing test setup, mailbox clearing, MSAL caches, or any hands-on testing activities.**

---

## ⚠️ E2E Test File (ALWAYS USE THIS)

**Standard test file:** `test_pdfs/E2E_Real_World_Test.xlsx`

This file is pre-configured with:
- 7 test properties (699 Industrial, 150 Trade Center, 2017 St. Josephs, 9300 Lottsford, 1 Randolph, 1800 Broad, 2525 Center West)
- Custom columns for testing: `Rail Access`, `Office %`, `Sprinkler Type`, `Building Condition Notes`
- Script sheet with outreach template (asks about parking)
- Broker emails split between `bp21harrison@gmail.com` and `baylor@manifoldengineering.ai`

**Column configuration during setup:**
| Column | Mode | Tests |
|--------|------|-------|
| Rail Access | Ask (Required) | Blocks closing if missing |
| Office % | Ask (Optional) | AI asks but doesn't block |
| Sprinkler Type | Accept Only | Extract if mentioned, never ask |
| Building Condition Notes | Note | Auto-append, never ask |

**DO NOT create new test Excel files. Always use this one.**

---

## Project Overview

**What is this?** An AI-powered commercial real estate email automation system that:
1. Sends personalized outreach emails to property brokers on behalf of tenant rep clients
2. Automatically processes broker replies using GPT-4o to extract property data
3. Populates Google Sheets with extracted information (SF, rent, docks, power, etc.)
4. Detects events requiring human attention (tours, negotiations, identity questions)
5. Maintains conversation threads until all required information is gathered

**Who uses it?** Tenant representatives (commercial real estate professionals who help businesses find warehouse/industrial space). They upload a spreadsheet of properties, configure email templates, and the system handles broker communication automatically.

**Business value:** Automates the tedious process of gathering property specifications from dozens of brokers simultaneously, freeing tenant reps to focus on high-value activities.

---

## Repository Structure

| Repository | Purpose | Tech Stack |
|------------|---------|------------|
| **EmailAutomation** (this repo) | Backend - email processing, AI extraction, sheet updates | Python, OpenAI, Microsoft Graph, Google Sheets API |
| **email-admin-ui** | Frontend - user dashboard, client management, campaign launch | React, Firebase, Tailwind CSS |

```
├── email_automation/           # Core Python modules
│   ├── ai_processing.py       # OpenAI extraction & event detection (THE BRAIN)
│   ├── processing.py          # Main pipeline: inbox → AI → sheet → notification
│   ├── email.py               # Outbox processing, email sending via Graph API
│   ├── sheets.py              # Google Sheets read/write operations
│   ├── sheet_operations.py    # NON-VIABLE divider, row movement
│   ├── messaging.py           # Firestore thread/message indexing
│   ├── notifications.py       # User notification creation
│   ├── column_config.py       # Dynamic column mapping system
│   ├── followup.py            # Automatic follow-up scheduling
│   └── clients.py             # Firebase/Google API client initialization
├── tests/                      # Comprehensive test suite
│   ├── standalone_test.py     # 25 AI extraction scenarios
│   ├── e2e_test.py            # Full pipeline tests with Scrub file
│   ├── campaign_lifecycle_test.py  # Multi-property campaign tests
│   └── conversations/         # Broker reply fixtures
├── main.py                     # Entry point: processes all users
├── app.py                      # Flask server for OAuth & debug APIs
└── Scrub Augusta GA.xlsx       # Test data file
```

---

## Accessing Outlook Inbox (IMPORTANT - READ THIS)

**DO NOT reinvent Outlook access. Use the helper script:**

```bash
# List inbox messages
python tests/outlook_helper.py inbox

# List inbox for specific user
python tests/outlook_helper.py inbox NO7lVYVp6BaplKYEfMlWCgBnpdh2

# Download attachments from a message
python tests/outlook_helper.py attachments <msg_id>

# List available user IDs
python tests/outlook_helper.py users
```

**Default user:** `NO7lVYVp6BaplKYEfMlWCgBnpdh2` (baylor.freelance@outlook.com)

**How it works internally:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BaylorH) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
