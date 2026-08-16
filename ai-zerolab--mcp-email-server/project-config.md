---
trigger: always_on
description: `mcp-email-server` is a Python MCP server that connects MCP clients to email
---

# Repository Overview

`mcp-email-server` is a Python MCP server that connects MCP clients to email
accounts through IMAP and SMTP.

Canonical repository:

- HTTPS: `https://github.com/Wh1isper/mcp-email-server`
- SSH: `git@github.com:Wh1isper/mcp-email-server.git`

Primary repository areas:

- `mcp_email_server/app.py` — FastMCP server, resources, tools, and tool visibility.
- `mcp_email_server/cli.py` — stdio, SSE, Streamable HTTP, UI, reset, and credential migration commands.
- `mcp_email_server/config.py` — TOML settings, environment composition, account models, and persistence.
- `mcp_email_server/keyring_store.py` — operating system keyring integration.
- `mcp_email_server/emails/` — IMAP and SMTP behavior and response models.
- `mcp_email_server/ui.py` / the replacement Web UI package — the local loopback management adapter and packaged static assets.
- `frontend/` — React/TypeScript management UI source and locked maintainer build.
- `plugins/`, `.agents/`, and `.claude-plugin/` — optional Codex/Claude Code plugin packaging, shared local MCP launch metadata, and safe non-secret setup guidance.
- `tests/` — unit, contract, integration, security, and browser-facing tests.
- `docs/` — user documentation published with MkDocs.
- `spec/` — unpublished normative architecture and product contracts kept as flat numbered documents.

## Project Conventions

- Support Python 3.11 and later.
- Use `uv` for dependency management and command execution.
- Follow the existing typed, asynchronous Python style.
- Prefer explicit types and `isinstance` checks over dynamic attribute checks.
- Use Pydantic models for configuration and structured responses.
- Keep MCP-facing descriptions accurate because clients derive tool schemas from them.
- Preserve the distinction between persistent TOML settings and the environment-composited runtime view.
- Treat credential storage, allowlists, attachment paths, and HTTP transport settings as security-sensitive behavior.
- Do not log, document, or commit real email credentials, API keys, message contents, or tokens.

## Current Architecture Direction

The revised Local Email App architecture under `spec/` is the normative target
currently being implemented. MCP stdio provides bounded mail workflows; CLI and
the embedded loopback-only React UI provide the managed management plane.
SQLite owns managed configuration and a rebuildable metadata projection. On
Linux and Windows, its private `managed_secret` table is the default
`SecretStore`; macOS uses the operating-system keyring. Application
services resolve only the selected account/role secret immediately before
provider construction. Legacy TOML/environment/keyring behavior remains an
explicit compatibility mode and import source, but MCP exposes no account or
credential management in either mode. Historical `add_email_account` is removed. The optional Codex/Claude plugin
launches the current published mail-only stdio server through a shared `.mcp.json`;
its semver is independent from the Python application, while its skill hands
account and credential setup to interactive CLI/UI without receiving secrets.
Attachment compatibility preserves the caller's exact destination behind
explicit policy and filesystem defenses.
Centralized limits apply to application and serialized results. MCP Apps, remote
UI, daemon, multi-user, hard purge, and cloud-service design remain out of scope.

## Development Workflow

Install the environment and pre-commit hooks:

```bash
make install
```

Before completing a change, run:

```bash
make check
make test
make docs-test
```

During development, focused checks are encouraged, but the full relevant suite
must pass before a change is considered complete.

Useful commands:

| Command                         | Description                                                  |
| ------------------------------- | ------------------------------------------------------------ |
| `uv run mcp-email-server stdio` | Run the local stdio MCP server.                              |
| `uv run mcp-email-server ui`    | Open the account configuration UI.                           |
| `make check`                    | Run lockfile, formatting, lint, type, and dependency checks. |
| `make test`                     | Run the test suite with coverage.                            |
| `make docs-test`                | Build the MkDocs site in strict mode.                        |
| `make docs`                     | Serve the documentation locally.                             |

## Documentation Requirements

Code and documentation must remain aligned.

- Every code change must include a review and corresponding update of the relevant documentation in the same change.
- User-visible behavior changes must always update the appropriate page under `docs/`; internal changes must still keep docstrings and developer guidance accurate.
- Configuration fields or environment variables require updates to `docs/configuration.md` and, when security-sensitive, `docs/security.md`.
- CLI commands, arguments, transport defaults, or HTTP security behavior require updates to `docs/transports.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ai-zerolab/mcp-email-server](https://github.com/ai-zerolab/mcp-email-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
