---
trigger: always_on
description: This file provides context for AI assistants (Claude, etc.) working on this project.
---

# CLAUDE.md - AI Assistant Guidelines

This file provides context for AI assistants (Claude, etc.) working on this project.

## Project Overview

**ARC-1** is a TypeScript MCP (Model Context Protocol) server for SAP ABAP Development Tools (ADT). It provides 12 intent-based tools (SAPRead, SAPSearch, SAPWrite, SAPActivate, SAPNavigate, SAPQuery, SAPTransport, SAPGit, SAPContext, SAPLint, SAPDiagnose, SAPManage) for use with Claude and other MCP-compatible LLMs.

Distributed as an npm package (`arc-1`) and Docker image (`ghcr.io/marianfoo/arc-1`).

## Design Principles

1. **Centralized admin control** — Runs as a managed service, not on developer laptops. Admins configure positive opt-in safety gates (writes, data preview, SQL, package allowlists, transport/Git writes, deny actions) per instance. Every tool call is audited with user identity. Per-user JWT scopes can restrict further but never expand beyond server config.

2. **Per-user SAP identity** — Principal propagation maps each MCP user to their own SAP user via BTP Destination Service + Cloud Connector. SAP's native authorization (S_DEVELOP, package checks) applies per user. No shared service accounts.

3. **Token-efficient tool design** — 12 intent-based tools (~5K schema tokens) instead of 200+ endpoints. Hyperfocused mode: 1 tool (~200 tokens). Method-level surgery (95% reduction) and context compression (7-30x) keep responses within tight context windows. This is the difference between working and not working on mid-tier LLMs (GPT-4o-mini, Copilot Studio).

4. **BTP-native deployment** — First-class BTP CF support: Destination Service, Cloud Connector, XSUAA OAuth, BTP Audit Log Service. Also deployable as Docker or npm. Local stdio mode for development.

5. **Multi-client, vendor-neutral** — Standard MCP protocol. Three auth modes coexist: XSUAA OAuth + Entra ID OIDC + API key. Same instance serves Claude, Copilot Studio, VS Code Copilot, Gemini CLI, Cursor.

6. **Safe defaults, opt-in power** — Read-only by default. Free SQL blocked. Package allowlist defaults to `$TMP`. Writing to transportable packages requires explicit config. Everything forbidden until the admin allows it.

## Quick Reference

### Build & Test

```bash
npm ci                          # Install dependencies
npm run build                   # TypeScript → dist/
npm test                        # Unit tests
npm run test:watch              # Unit tests (watch mode)
npm run typecheck               # Type check
npm run lint                    # Lint
npm run test:integration        # Integration tests (needs SAP credentials)
npm run test:integration:crud   # CRUD lifecycle tests (needs SAP credentials)
npm run test:coverage           # Unit tests with coverage (informational)
npm run test:coverage-report    # Coverage summary (Markdown)
npm run dev                     # Dev mode (stdio)
npm run dev:http                # Dev mode (HTTP Streamable)
# BTP tests (local only — needs service key + browser login):
TEST_BTP_SERVICE_KEY_FILE=~/.config/arc-1/btp-abap-service-key.json npm run test:integration:btp
TEST_BTP_SERVICE_KEY_FILE=~/.config/arc-1/btp-abap-service-key.json npm run test:integration:btp:smoke
```

### Configuration (Priority: CLI > Env > .env > Defaults)

```bash
# Using environment variables
SAP_URL=http://host:50000 SAP_USER=user SAP_PASSWORD=pass npm run dev

# Using .env file (copy .env.example to .env)
npm run dev
```

| Variable / Flag | Description |
|-----------------|-------------|
| `SAP_URL` / `--url` | SAP system URL (e.g., `http://host:50000`) |
| `SAP_USER` / `--user` | SAP username |
| `SAP_PASSWORD` / `--password` | SAP password |
| `SAP_CLIENT` / `--client` | SAP client number (default: 100) |
| `SAP_LANGUAGE` / `--language` | SAP language (default: EN) |
| `SAP_INSECURE` / `--insecure` | Skip TLS verification (default: false) |
| `SAP_TRANSPORT` / `--transport` | MCP transport: `stdio` (default) or `http-streamable` |
| `ARC1_PORT` / `--port` | HTTP server port (default: `8080`). Simpler alternative to `ARC1_HTTP_ADDR` when only the port needs to change |
| `ARC1_HTTP_ADDR` / `--http-addr` | HTTP server bind address (default: `0.0.0.0:8080`). Use when you need to change both host and port |
| `SAP_ALLOW_WRITES` / `--allow-writes` | Enable object mutations (default: `false`). Required for transport/git writes too. |
| `SAP_ALLOW_DATA_PREVIEW` / `--allow-data-preview` | Enable named table preview (default: `false`) |
| `SAP_ALLOW_FREE_SQL` / `--allow-free-sql` | Enable freestyle SQL (default: `false`) |
| `SAP_DENY_ACTIONS` / `--deny-actions` | Fine-grained per-action denial. Grammar: `Tool`, `Tool.action`, `Tool.glob*`. Inline CSV or file path. See [authorization.md](docs_page/authorization.md#advanced-deny-actions). |
| `SAP_ALLOWED_PACKAGES` / `--allowed-packages` | Restrict write operations to packages (default: `$TMP`; supports wildcards: "Z*"). Reads never package-gated. |
| `SAP_ALLOW_TRANSPORT_WRITES` / `--allow-transport-writes` | Enable transport mutations (default: `false`). Requires `SAP_ALLOW_WRITES=true`. |
| `SAP_ALLOW_GIT_WRITES` / `--allow-git-writes` | Enable git mutations (default: `false`). Requires `SAP_ALLOW_WRITES=true`. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marianfoo/arc-1](https://github.com/marianfoo/arc-1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
