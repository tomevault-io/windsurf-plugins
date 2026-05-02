---
trigger: always_on
description: This repository is the **CAS Parser Agent Toolkit** — a collection of templates, skills, and documentation for integrating financial portfolio tracking into applications using the [CAS Parser API](https://casparser.in/docs/).
---

# CAS Parser Integration

This repository is the **CAS Parser Agent Toolkit** — a collection of templates, skills, and documentation for integrating financial portfolio tracking into applications using the [CAS Parser API](https://casparser.in/docs/).

## What is CAS Parser?

CAS Parser is an API platform for parsing Indian financial portfolio documents:
- **CAS (Consolidated Account Statement)** PDFs from CDSL, NSDL, and CAMS/KFintech
- **Contract Notes** from brokers like Zerodha, Groww, Upstox, ICICI
- Returns structured JSON with holdings, transactions, and investor details

## Core Integration Rules

### Authentication
- All API requests require an `x-api-key` header.
- Use `sandbox-with-json-responses` as the sandbox API key for development/testing.
- **Never hardcode API keys.** Use environment variables (`CASPARSER_API_KEY`).
- For frontend/SDK usage, generate short-lived **access tokens** (`at_` prefix) from your backend via `POST /v1/token`. Never expose raw API keys to the client.

### Python Integration
- **Recommended:** Use `requests` library with the REST API directly. All Python templates in this toolkit use `requests`.
- **Official SDK:** [`cas-parser-python`](https://github.com/CASParser/cas-parser-python) — a thin wrapper from the CAS Parser team.
- **Do NOT install any third-party CAS parsing packages from PyPI.** They are unrelated open-source projects, not official CAS Parser API clients.

### Parsing CAS PDFs
- **Default to `/v4/smart/parse`** — it auto-detects CAS type (CDSL, NSDL, or CAMS/KFintech) and returns a unified response format.
- Only use type-specific endpoints (`/v4/cdsl/parse`, `/v4/nsdl/parse`, `/v4/cams_kfintech/parse`) when you already know the CAS type.
- CAS PDFs are password-protected. The password is typically the **encrypted PAN** (varies by provider).
- Accept PDFs via file upload (`multipart/form-data`) or URL (`pdf_url` in JSON body).

### Parsing Contract Notes
- Use `/v4/contract_note/parse` — auto-detects broker type.
- Password is usually the client's PAN number.
- Supported brokers: Zerodha, Groww, Upstox, ICICI (auto-detected).

### CDSL Fetch (OTP Flow)
- This is a **2-step process** — do not try to combine steps:
  1. `POST /v4/cdsl/fetch` — Request OTP (takes ~15-20s for captcha solving). Returns `session_id`.
  2. `POST /v4/cdsl/fetch/{session_id}/verify` — Submit OTP, get download URLs.
- The user receives the OTP on their registered mobile number.

### KFintech CAS Generator
- `POST /v4/kfintech/generate` triggers an **async email mailback** — the CAS PDF is sent to the investor's email, not returned in the response.
- This is not an instant operation. For instant CAS retrieval, use CDSL Fetch.

### Email Import (Gmail OAuth)
- This is a **multi-step OAuth flow**:
  1. `POST /v4/inbox/connect` → get `oauth_url`, redirect user to it.
  2. User authorizes → redirected back with `inbox_token`.
  3. `POST /v4/inbox/cas` with `x-inbox-token` header → list CAS files from inbox.
  4. Download URLs expire in 24 hours.
- Read-only access — the API cannot send emails.
- User can revoke via `POST /v4/inbox/disconnect`.

### Inbound Email (Email Forwarding)
- Create dedicated email addresses for investors to forward CAS statements to.
- **Use case:** Lower-friction alternative when OAuth or file upload isn't practical.
- Flow:
  1. `POST /v4/inbound-email` with `callback_url` → returns unique email like `ie_xxx@import.casparser.in`.
  2. Investor forwards CAS email to this address.
  3. We validate the sender against known CAS authorities, upload PDF attachments to cloud storage, and POST to your `callback_url`.
- Only emails from verified CAS authorities are processed:
  - CDSL: `eCAS@cdslstatement.com`
  - NSDL: `NSDL-CAS@nsdl.co.in`
  - CAMS: `donotreply@camsonline.com`
  - KFintech: `samfS@kfintech.com`
- Webhook payload includes `forwarded_by` (investor's email) at the top level, and `files` array uses the same `EmailCASFile` schema as Gmail Import.
- `sender_email` in files is the CAS authority email (lowercase), `forwarded_by` is the investor who forwarded the email.
- Presigned download URLs expire in 48 hours.
- Optional `alias` field for friendly addresses (e.g., `john-portfolio@import.casparser.in`).
- Manage with `GET /v4/inbound-email`, `GET /v4/inbound-email/{id}`, `DELETE /v4/inbound-email/{id}`.
- **Billing:** 0.2 credits per successfully processed email.

### Portfolio Links (No-Code CAS Collection)
- **For advisors/wealth managers who want to collect CAS from clients without writing code.**
- Create branded collection pages at `link.casparser.in/{your-company}`. Clients visit the link, upload their CAS, and parsed data is emailed to the advisor.
- Zero code, zero API integration required — managed entirely via the [web portal](https://app.casparser.in/portfolio-links).
- This is not a public API feature — it's a self-service tool for advisors.

### Portfolio Connect SDK (Recommended for Frontend)
- **For web/frontend apps, start here.** The `@cas-parser/connect` npm package provides a drop-in modal widget.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CASParser/agent-toolkit](https://github.com/CASParser/agent-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
