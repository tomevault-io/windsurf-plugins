---
trigger: always_on
description: Complete Zoho Mail integration with OAuth2, REST API (5-10x faster), Clawdbot /email commands, HTML emails, attachments, and batch operations. Security-hardened against path traversal and command injection. Perfect for email automation and workflows.
---


# Zoho Email Integration

**v2.2.9** - Complete Zoho Mail integration with OAuth2 authentication, REST API backend (5-10x faster than IMAP/SMTP), and **Clawdbot extension with /email commands for Telegram/Discord**. **Security-hardened** against path traversal and command injection. Supports HTML emails, attachments, batch operations, and advanced automation workflows.

Choose your authentication: OAuth2 (recommended, secure) or app password (simple setup).

## 🔄 Update to Latest Version

```bash
clawhub install zoho-email-integration --force
```

Or update all skills:
```bash
clawhub update
```

## 🔒 Security Notice (v2.2.5+)

**CRITICAL FIX:** Removed vulnerable JavaScript command handler. If you deployed `email-command.js` from the examples folder, update immediately:

```bash
# Re-download the secure handler
clawhub install zoho-email-integration --force
cp ~/.openclaw/skills/zoho-email-integration/examples/clawdbot-extension/email-command.js /your/deployment/path/
```

The vulnerable version used `execSync` with shell interpolation. The new version uses `spawn` with argument arrays to prevent command injection.

## ✨ Features

### 🔐 Authentication & Performance
- **OAuth2 authentication** - Secure token-based auth with automatic refresh
- **REST API backend** - 5-10x faster operations than IMAP/SMTP
- **Graceful fallback** - Automatically falls back to IMAP if REST API unavailable
- **App password support** - Simple alternative to OAuth2

### 📧 Email Operations
- **📥 Read emails** - Fetch from any folder (Inbox, Sent, Drafts, etc.)
- **🔍 Smart search** - Search by subject, sender, keywords with REST API speed
- **📊 Monitor inbox** - Real-time unread count for notifications
- **📤 Send emails** - Plain text or HTML with CC/BCC support
- **🎨 HTML emails** - Rich formatting with professional templates included
- **📎 Attachments** - Send and download file attachments

### ⚡ Batch & Bulk Operations
- **Batch operations** - Mark, delete, or move multiple emails efficiently
- **Bulk actions** - Search and act on hundreds of emails at once
- **Dry-run mode** - Preview actions before executing for safety

### 🔒 Security
- **No hardcoded credentials** - OAuth2 tokens or environment variables only
- **Automatic token refresh** - Seamless token renewal
- **Encrypted connections** - SSL/TLS for all operations

## 📦 Installation

```bash
clawhub install zoho-email-integration
```

**Requirements:**
- Python 3.x
- `requests` library (install: `pip3 install requests`)
- Zoho Mail account

## ⚙️ Setup

### 1. Get an App-Specific Password

**Important:** Don't use your main Zoho password!

1. Log in to Zoho Mail
2. Go to **Settings** → **Security** → **App Passwords**
3. Generate a new app password for "Clawdbot" or "IMAP/SMTP Access"
4. Copy the password (you'll need it next)

### 2. Configure Credentials

**Option A: Environment Variables**

Export your Zoho credentials:

```bash
export ZOHO_EMAIL="your-email@domain.com"
export ZOHO_PASSWORD="your-app-specific-password"
```

**Option B: Credentials File**

Create `~/.clawdbot/zoho-credentials.sh`:

```bash
#!/bin/bash
export ZOHO_EMAIL="your-email@domain.com"
export ZOHO_PASSWORD="your-app-specific-password"
```

Make it executable and secure:
```bash
chmod 600 ~/.clawdbot/zoho-credentials.sh
```

Then source it before running:
```bash
source ~/.clawdbot/zoho-credentials.sh
```

### 3. Test Connection

```bash
python3 scripts/zoho-email.py unread
```

Expected output:
```json
{"unread_count": 5}
```

## 🚀 Usage

All commands require credentials set via environment variables.

### Quick commands (common tasks)

```bash
# Diagnose setup (recommended first step)
python3 scripts/zoho-email.py doctor

# Unread count (great for briefings)
python3 scripts/zoho-email.py unread

# Search inbox
python3 scripts/zoho-email.py search "invoice"

# Get a specific email (folder + id)
python3 scripts/zoho-email.py get INBOX <id>

# Send a simple email
python3 scripts/zoho-email.py send recipient@example.com "Subject" "Body text"

# Empty Spam (safe by default: DRY RUN)
python3 scripts/zoho-email.py empty-spam
# Execute for real
python3 scripts/zoho-email.py empty-spam --execute

# Empty Trash (safe by default: DRY RUN)
python3 scripts/zoho-email.py empty-trash
# Execute for real
python3 scripts/zoho-email.py empty-trash --execute
```

### Send HTML Emails

Send rich, formatted HTML emails with multipart/alternative support (both HTML and plain text versions):

**CLI Command:**
```bash
# Send HTML from a file
python3 scripts/zoho-email.py send-html recipient@example.com "Newsletter" examples/templates/newsletter.html

# Send HTML from inline text
python3 scripts/zoho-email.py send-html recipient@example.com "Welcome" "<h1>Hello!</h1><p>Welcome to our service.</p>"

# Preview HTML email before sending
python3 scripts/zoho-email.py preview-html examples/templates/newsletter.html
```

**Python API:**
```python
from scripts.zoho_email import ZohoEmail

zoho = ZohoEmail()

# Method 1: Send HTML with auto-generated plain text fallback
zoho.send_html_email(
    to="recipient@example.com",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [briansmith80/clawdbot-zoho-email](https://github.com/briansmith80/clawdbot-zoho-email) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
