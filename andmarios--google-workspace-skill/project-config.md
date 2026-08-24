---
trigger: always_on
description: This file provides context for Claude when working on this codebase.
---

# Claude Code Development Guide

This file provides context for Claude when working on this codebase.

## Project Overview

This is a **Claude Code skill** that provides Google Workspace integration. The skill exposes a CLI (`gws-cli`) that Claude invokes to interact with Google APIs.

**Key distinction:**
- `SKILL.md` - Instructions for Claude when **using** the skill (overview, navigation)
- `reference/*.md` - Service-specific API documentation (docs, sheets, slides, etc.)
- `SKILL-advanced.md` - Design best practices, content creation, API efficiency
- `CLAUDE.md` - Instructions for Claude when **developing** the skill (this file)

## Architecture

```
src/gws/
├── cli.py              # Main Typer app, auth/config/account commands
├── config.py           # Service config + multi-account registry
├── crypto.py           # Fernet encryption at rest (machine-derived key, HKDF)
├── context.py          # Runtime active account state
├── output.py           # JSON output formatting (output_success, output_error)
├── exceptions.py       # Exit codes (0-4)
├── auth/
│   ├── oauth.py        # OAuth loopback flow (ports 8080-8099), account-aware token paths
│   └── scopes.py       # Per-service Google API scopes
├── services/
│   ├── base.py         # BaseService class (handles auth, builds API client, account context)
│   ├── drive.py        # Google Drive operations
│   ├── docs.py         # Google Docs operations
│   ├── sheets.py       # Google Sheets operations
│   ├── slides.py       # Google Slides operations
│   ├── gmail.py        # Gmail operations
│   ├── calendar.py     # Google Calendar operations
│   ├── contacts.py     # People API operations
│   └── convert.py      # Markdown → Docs/Slides/PDF converter
├── commands/
│   ├── _account.py     # Shared --account/-a Typer callback for all service commands
│   ├── docs.py         # Typer CLI commands (mirror services/)
│   └── ...
└── utils/
    ├── colors.py       # Hex color → RGB conversion for Sheets/Slides
    ├── diagrams.py     # Kroki API diagram rendering
    └── markdown.py     # Markdown parser for slides conversion
```

## Key Patterns

### Service Structure

Each service follows this pattern:

```python
class SomeService(BaseService):
    SERVICE_NAME = "some_api"  # Google API name
    VERSION = "v1"             # API version

    def operation(self, ...) -> dict[str, Any]:
        try:
            result = self.service.resource().method(...).execute()
            output_success(operation="service.op", ...)
            return result
        except HttpError as e:
            output_error(error_code="API_ERROR", operation="service.op", message=...)
            raise SystemExit(ExitCode.API_ERROR)
```

### Output Format

All commands output JSON via `output_success()` or `output_error()`:

```python
# Success
output_success(operation="docs.read", document_id=doc_id, content=text)

# Error
output_error(error_code="NOT_FOUND", operation="docs.read", message="Document not found")
```

### CLI Commands

Commands are thin wrappers that parse args and call service methods:

```python
@app.command("read")
def read_document(document_id: Annotated[str, typer.Argument(...)]) -> None:
    service = DocsService()
    service.read_document(document_id=document_id)
```

## Running and Testing

```bash
# Run CLI directly
uv run gws-cli --help
uv run gws-cli docs read <doc_id>

# Run tests
uv run pytest

# Type checking
uv run mypy src/

# Linting
uv run ruff check .
```

## Credentials

Stored in `~/.config/gws-cli/` with encryption at rest:
- `client_secret.json.enc` - OAuth client credentials (encrypted, imported via `gws-cli auth import-credentials`)
- `token.json.enc` - Access token (encrypted, auto-generated during OAuth flow)
- `gws_config.json` - Configuration (plaintext — contains `encryption_salt` + enabled services, Kroki URL, accounts registry)

### Encryption at Rest

Secret files (tokens, credentials) are encrypted using Fernet symmetric encryption. The key is derived at runtime from machine-specific identifiers and never stored on disk.

- **Module**: `src/gws/crypto.py`
- **Key derivation**: `machine_id + username + app_id` → HKDF-SHA256 → Fernet key
- **File convention**: `foo.json` → encrypted as `foo.json.enc`
- **Auto-migration**: plaintext files are automatically encrypted on first load
- **Opt-out**: Set `GWS_ENCRYPTION=none` to disable encryption entirely
- **Salt**: Auto-generated, stored in `gws_config.json` (plaintext config stays unencrypted)

CLI commands for managing encrypted credentials:
- `gws-cli auth import-credentials <path>` - Import and encrypt `client_secret.json` from Google Cloud Console

### Multi-Account Storage

When multi-account mode is active, per-account data is stored under `accounts/`:

```
~/.config/gws-cli/
├── client_secret.json              # Shared OAuth client (unchanged)
├── token.json                      # Legacy token (kept, used when no accounts)
├── gws_config.json                 # Global config + accounts registry
└── accounts/
    ├── work/
    │   ├── token.json              # Account-specific token
    │   └── config.json             # Per-account overrides (optional)
    └── personal/
        └── token.json
```

## Multi-Account Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andmarios/google-workspace-skill](https://github.com/andmarios/google-workspace-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
