---
trigger: always_on
description: This document is intended for AI development tools and future maintainers to understand the project structure and design decisions.
---

# AGENTS.md - Project Structure and Purpose

This document is intended for AI development tools and future maintainers to understand the project structure and design decisions.

## Project Purpose

`gmail-ai-unsub` is a Python CLI tool that uses Large Language Models (LLMs) to identify marketing emails in Gmail and automatically unsubscribe from them. The tool combines:

1. **Email Classification**: LLM-based analysis to identify marketing/promotional emails
2. **Unsubscribe Automation**: Multiple strategies for unsubscribing (headers, mailto, browser automation)

## Architecture Overview

### Core Components

```
src/gmail_ai_unsub/
├── cli.py              # Click-based CLI entry point
├── config.py           # TOML configuration loader
├── storage.py          # JSON state persistence
├── gmail/              # Gmail API integration
│   ├── auth.py         # OAuth2 authentication
│   ├── client.py       # API client with retry logic
│   └── labels.py       # Label management
├── classifier/         # Email classification
│   └── email_classifier.py  # LangChain-based LLM classifier
└── unsubscribe/        # Unsubscribe automation
    ├── extractor.py    # Extract unsubscribe links/headers
    ├── email_unsub.py  # Header-based unsubscribe (RFC 8058)
    └── browser_agent.py # Browser automation with browser-use
```

### Design Decisions

#### 0. Build System: uv-build

- **Build Backend**: `uv-build` (specified in `pyproject.toml`)
- **Why**: Explicitly chosen by project maintainer
- **Important**: **DO NOT** change to another build system (hatchling, setuptools, etc.) without explicit user approval
- **Rationale**: The build system is a fundamental project decision that affects packaging, distribution, and CI/CD

#### 1. LangChain for LLM Integration
- **Why**: Consistent API across providers (Gemini, Claude, OpenAI)
- **Why**: Required by `browser-use` for browser automation
- **Why**: Built-in structured output parsing (Pydantic integration)
- **Trade-off**: Adds dependency overhead, but simplifies multi-provider support

#### 2. browser-use for Browser Automation
- **Why**: Modern, actively maintained, integrates with LangChain
- **Why**: Supports vision models (Gemini 2.5 Computer Use, GPT-4V)
- **Why**: Handles complex unsubscribe pages with dark patterns
- **Alternative considered**: Stagehand, but browser-use has better LangChain integration

#### 3. TOML Configuration
- **Why**: Human-readable, supports nested structures
- **Why**: Standard format for Python projects
- **Why**: Easy to template and document

#### 4. JSON State Storage
- **Why**: Simple, human-readable, no database needed
- **Why**: Stores unsubscribe links and processing status
- **Why**: Easy to inspect and debug

#### 5. Two-Phase Workflow
- **Phase 1 (Scan)**: Identify and label marketing emails
- **Phase 2 (Unsubscribe)**: Process labeled emails for unsubscription
- **Why**: Allows user review before unsubscribing
- **Why**: Separates concerns (classification vs. action)

### Key Flows

#### Email Classification Flow

```
1. Gmail API → Fetch messages (metadata format for quota efficiency)
2. Extract subject, from, body
3. LangChain → Send to LLM with user-defined prompts
4. LLM → Returns structured result (is_marketing, confidence, reason)
5. If marketing → Apply label, extract unsubscribe link
6. Store unsubscribe link in state.json
```

#### Unsubscribe Flow

```
1. Load emails with marketing label
2. For each email:
   a. Check state.json for cached unsubscribe link
   b. If not found, extract from message (header or body)
   c. Try header-based unsubscribe (RFC 8058 one-click or mailto)
   d. If header fails and URL exists → Browser automation
   e. Update labels and state based on result
```

### Gmail API Considerations

- **Quota Management**: Uses `format=metadata` when possible (5 units vs. full message)
- **Rate Limiting**: Exponential backoff for 429 errors
- **OAuth Scopes**: Requires `gmail.readonly`, `gmail.modify`, `gmail.send`
  - `gmail.readonly`: Read email messages and metadata for classification
  - `gmail.modify`: Add/remove labels on messages
  - `gmail.send`: Send unsubscribe emails (for mailto: links)

#### OAuth Credentials Management

The tool uses a flexible credential management system suitable for open-source distribution.

**Approach:**
- **Development/CI**: Credentials loaded from environment variables (`GMAIL_CLIENT_ID`, `GMAIL_CLIENT_SECRET`)
  - Use `.env` file (gitignored) for local development
  - Use GitHub Secrets for CI builds
- **PyPI Wheels**: Credentials injected at build time via `BUILD_GMAIL_CLIENT_ID` and `BUILD_GMAIL_CLIENT_SECRET`
  - Build script: `scripts/build-with-credentials.sh`
  - Credentials are embedded in the wheel but not in source tree
- **Source Tree**: Contains testing/placeholder credentials (not production secrets)
- **User Override**: Users can provide their own `credentials.json` for custom OAuth apps

**Security Model:**
- For native/desktop apps, Google treats them as "public clients"
- The client_secret cannot be kept truly secret in open-source apps
- Security comes from:
  - PKCE (Proof Key for Code Exchange) - implemented via `google_auth_oauthlib`
  - User consent (OAuth flow)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zbowling/gmail-ai-unsub](https://github.com/zbowling/gmail-ai-unsub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
