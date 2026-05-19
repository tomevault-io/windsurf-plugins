---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the official **Mailtrap OpenAPI Specifications** repository. It contains OpenAPI 3.1 specification files for all Mailtrap API services. These specs are:
- Publicly accessible for client generation and API exploration
- Auto-fetched by GitBook to render interactive API documentation at https://docs.mailtrap.io/developers
- Used as the source of truth for Mailtrap API documentation

## Repository Structure

```
mailtrap-openapi/
├── specs/                    # OpenAPI specification files
│   ├── contacts.openapi.yml
│   ├── email-sending.openapi.yml
│   ├── email-sending-bulk.openapi.yml
│   ├── email-sending-transactional.openapi.yml
│   ├── account-management.openapi.yml
│   ├── sandbox.openapi.yml
│   ├── sandbox-sending.openapi.yml
│   └── templates.openapi.yml
├── .claude/
│   └── skills/
│       └── gitbook-assistant/    # GitBook formatting skill
└── README.md
```

## OpenAPI Specifications

### Available Specs

| Spec File | Description | Base URL |
|-----------|-------------|----------|
| `account-management.openapi.yml` | Account, users, permissions, billing | `https://mailtrap.io` |
| `contacts.openapi.yml` | Contact management | `https://mailtrap.io` |
| `email-sending.openapi.yml` | Combined email API reference | Varies by operation |
| `email-sending-bulk.openapi.yml` | Bulk/marketing email sending | `https://bulk.api.mailtrap.io` |
| `email-sending-transactional.openapi.yml` | Transactional email sending | `https://send.api.mailtrap.io` |
| `sandbox-sending.openapi.yml` | Sandbox send & batch email | `https://sandbox.api.mailtrap.io` |
| `sandbox.openapi.yml` | Email Sandbox/testing | `https://mailtrap.io` |
| `templates.openapi.yml` | Email templates | `https://mailtrap.io` |

### Base URL Rules

**CRITICAL: Never change base URLs in OpenAPI specs. The correct base URLs are:**

- **Transactional emails**: `https://send.api.mailtrap.io`
- **Bulk emails**: `https://bulk.api.mailtrap.io`
- **All other APIs**: `https://mailtrap.io`

## Link Formatting

### Documentation Links

**All links in OpenAPI specs MUST be absolute URLs pointing to docs.mailtrap.io**

Do NOT use:
- Relative links
- Links to help.mailtrap.io (legacy HelpScout)
- Links to app.gitbook.com internal URLs in public-facing descriptions

**Correct format:**
```yaml
description: |
  Learn more about [Sending Domain Setup](https://docs.mailtrap.io/email-api-smtp/setup/sending-domain).
```

### Link Reference Mapping

A mapping file `helpscout_to_gitbook_redirects.csv` is available locally (not committed) that maps old HelpScout URLs to new GitBook URLs. Use this when updating legacy links:

| Old URL Pattern | New URL Pattern |
|-----------------|-----------------|
| `help.mailtrap.io/article/*` | `docs.mailtrap.io/*` |

**Common documentation links:**

| Topic | URL |
|-------|-----|
| Getting Started | `https://docs.mailtrap.io/getting-started` |
| API Integration | `https://docs.mailtrap.io/email-api-smtp/setup/api-integration` |
| SMTP Integration | `https://docs.mailtrap.io/email-api-smtp/setup/smtp-integration` |
| Sending Domain Setup | `https://docs.mailtrap.io/email-api-smtp/setup/sending-domain` |
| Bulk Stream | `https://docs.mailtrap.io/email-api-smtp/setup/bulk-stream` |
| Email Templates | `https://docs.mailtrap.io/email-marketing/campaigns/email-templates` |
| Email Logs | `https://docs.mailtrap.io/email-api-smtp/analytics/logs` |
| Email Sandbox | `https://docs.mailtrap.io/getting-started/email-sandbox` |
| API Tokens | `https://docs.mailtrap.io/email-sandbox/setup/api-tokens` |
| FAQs | `https://docs.mailtrap.io/email-api-smtp/help/faqs` |

### Contact Information

Update contact URLs in spec `info` blocks:
```yaml
info:
  contact:
    name: Mailtrap Support
    url: 'https://docs.mailtrap.io'  # NOT help.mailtrap.io
    email: support@mailtrap.io
```

## GitBook Formatting in OpenAPI

OpenAPI descriptions support GitBook markdown syntax. Use the **gitbook-assistant skill** for proper formatting.

### Supported GitBook Blocks

Use these blocks in `description` fields:

```yaml
description: |
  {% hint style="info" %}
  This is an informational callout.
  {% endhint %}

  {% hint style="warning" %}
  This is a warning callout.
  {% endhint %}

  {% tabs %}
  {% tab title="Option A" %}
  Content for option A
  {% endtab %}
  {% tab title="Option B" %}
  Content for option B
  {% endtab %}
  {% endtabs %}
```

### Critical Formatting Rules

1. **Always close blocks**: `{% hint %}` → `{% endhint %}`, `{% tab %}` → `{% endtab %}`
2. **Proper nesting**: Tabs cannot be inside details blocks
3. **No orphaned tags**: Every opening tag needs a closing tag

## Code Samples (x-codeSamples)

### SDK Priority Order

Include code samples in this order:
1. cURL (shell)
2. Node.js (JavaScript)
3. PHP
4. Python
5. Ruby
6. .NET (C#)
7. Java

### Code Sample Format

```yaml
x-codeSamples:
  - lang: javascript
    label: Node.js
    source: |
      import { MailtrapClient } from "mailtrap";

      const client = new MailtrapClient({
        token: process.env.MAILTRAP_API_KEY
      });

      // SDK-specific code here
  - lang: php

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mailtrap/mailtrap-openapi](https://github.com/mailtrap/mailtrap-openapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
