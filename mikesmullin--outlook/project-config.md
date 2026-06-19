---
trigger: always_on
description: interact w/ Microsoft Office (O365) Outlook Email inbox
---


# Outlook Emails

## Overview

The `outlook-email` command is installed globally.

It is a dual-mode email management system for Office 365 (Microsoft Outlook) with offline analysis capabilities. 

It provides:

1. **Online Mode**: Pull emails from Outlook to local Markdown storage
2. **Offline Mode**: Query and manage the stored email database without Outlook connectivity

The system is designed for AI agents to integrate email processing into workflows, supporting:
- Batch email ingestion with deduplication
- Email marking (read/unread) with offline metadata
- Pattern analysis on stored emails
- Integration with other tools and scripts

## Core Concepts

### Email IDs
- **Outlook ID**: Long base64-encoded identifier from Microsoft Graph API
- **SHA1 Hash**: 40-character hex hash of Outlook ID, used as filename
- **Short ID**: First 6 characters of hash (e.g., `6498ce`), used for Git-like partial matching

Example:
```
Outlook ID: AQMkAGJjZGY31MGViLTEYi1iYWQ2LTBjNDBjZjAzYmE3MgBGAA...
SHA1 Hash:  6498cec18d676f0328ff649bf933e7ec3c0adb2b
Short ID:   6498ce
```

### Storage Format
Each email is stored as a Markdown file in `storage/` with YAML front matter containing metadata, and the HTML body in a code block:

```markdown
---
id: 'AQMkAGJ...'
subject: 'Project Status Update'
from:
  emailAddress:
    name: 'Alice Smith'
    address: 'asmith@company.com'
toRecipients:
  - emailAddress:
      name: 'Team'
      address: 'team@company.com'
receivedDateTime: '2026-01-05T10:30:00Z'
isRead: false
webLink: 'https://outlook.office365.com/owa/?ItemID=...'
body:
  contentType: html
_stored_id: '6498cec18d676f08ff64932bf93e7ec33c0adb2b'
_stored_at: '2026-01-05T18:05:13.476Z'
offline:
  read: true              # Custom offline metadata
  readAt: '2026-01-05T18:06:00.000Z'
---

# Project Status Update

```html
<html>
<head>...</head>
<body>
<p>Email content here...</p>
</body>
</html>
```
```

### ID Matching (Git-style)
All CLI commands support partial IDs. The system matches the longest unique prefix:

```bash
# These all refer to the same email:
outlook-email view 6498cec18d676f08ff64932bf93e7ec33c0adb2b  # Full (40 chars)
outlook-email view 6498cec18d676f08                            # 16 chars
outlook-email view 6498ce                                      # 6 chars (short)
outlook-email view 6498                                        # 4 chars (if unique)
outlook-email view 6498cec18d676f08ff64932bf93e7ec33c0adb2b.md  # Filename format
```

Error on ambiguity:
```
Error: Ambiguous ID "62". Matches: 62e8e2d5adb20b15..., 62b19cb17ec4628a...
```

## Online Mode: Fetching Emails from Outlook

### Command: `outlook-email pull`

**Purpose**: Fetch unread emails from Outlook, store locally as Markdown files, mark as read/processed in Outlook.

> **IMPORTANT**: When instructed to fetch more emails, use this command. **Always fetch exactly one email at a time** (`--limit 1`) unless specifically directed to pull more. This ensures controlled processing and avoids overwhelming the local storage with unreviewed emails.

**Command**:
```bash
outlook-email pull --since <date> [--limit N]
```

**Parameters**:
- `--since <date>`: Required. Fetch emails received on/after this date
  - Formats: `YYYY-MM-DD`, `yesterday`, `"7 days ago"`, `"1 day ago"`
- `--limit <n>`: Optional. Stop after processing N emails (default: no limit)

**Behavior**:
1. Fetches all unread emails from inbox since date
2. Paginates through results (50 per request)
3. Stops when reaching emails older than date
4. Skips already-stored files (deduplication via SHA1)
5. Stores each new email as Markdown under `storage/<id>.md`
6. Marks processed emails as read in Outlook
7. Moves processed emails to "Processed" folder in Outlook
8. Updates `webLink` with the new permalink after move
9. Prints progress: `✓ Stored: <id>+<subject...>`

**Examples**:

```bash
# RECOMMENDED: Pull exactly 1 new email (safest, most controlled)
outlook-email pull --since yesterday --limit 1

# Pull from specific date, one at a time
outlook-email pull --since 2026-01-01 --limit 1

# Pull from last week, one at a time
outlook-email pull --since "7 days ago" --limit 1

# Pull multiple emails (only when specifically instructed)
outlook-email pull --since yesterday --limit 5

# Pull all unread from past week (use with caution)
outlook-email pull --since "7 days ago"
```

**Output Example**:
```
Fetching unread emails since: 2026-01-05
Processing limit: 1
Found 14 unread emails.
✓ Stored: (71c95a7e429ff98a+NOTICE: LDAP Password Expiration)
  → Marking as read...
  → Moving to Processed folder...
  ✓ Updated in Outlook (webLink updated)

Reached processing limit of 1. Stopping.

Summary:
  Available:  14
  Processed:  1
  Written:    1
  Skipped:    0
```

**Best Practices**:
- Always use `--limit 1` unless you need to bulk import
- Use `--since yesterday` or `--since "1 day ago"` for recent emails
- After pulling, use `outlook-email list` to see the new email
- The `webLink` field in the stored file opens the email directly in Outlook

## Offline Mode: Analysis & Metadata

### Local Query & Management

All `outlook-email` commands work purely offline, reading/writing YAML files in `storage/`.

### Command: `outlook-email list [OPTIONS]`

**Purpose**: List emails with filtering and display

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikesmullin/outlook](https://github.com/mikesmullin/outlook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
