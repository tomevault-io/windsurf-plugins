---
trigger: always_on
description: This file is read by AI coding agents (Claude Code, Cursor, Copilot, etc.) by convention. It documents how an agent should help a user integrate Authplane into the user's MCP server, agent, or operations workflow.
---

# AGENTS.md — Agent Integration Guide

This file is read by AI coding agents (Claude Code, Cursor, Copilot, etc.) by convention. It documents how an agent should help a user integrate Authplane into the user's MCP server, agent, or operations workflow.

The 98% case: a user has an MCP server (or is writing one) and wants Authplane to handle auth. The 2% case: a contributor is changing authserver itself — for that, see [CONTRIBUTING.md](CONTRIBUTING.md) and [docs/contribute/](docs/contribute/).

## Hard rules

1. **Do not infer flag names, env-var names, DTO fields, or YAML keys from training data or older code samples.** They drift. The four generated reference files below are the only source of truth.
2. **Do not hand-edit `docs/reference/cli.md`, `http-api.md`, `env-vars.md`, or `configuration.md`** — they regenerate from source via `make docs-gen` (run from the authserver repo root). Hand edits fail CI.
3. **Use the published SDKs** (PyPI / npm / Go modules), not in-tree code. Sibling repos: [go-sdk](https://github.com/authplane/go-sdk), [ts-sdk](https://github.com/authplane/ts-sdk), [python-sdk](https://github.com/authplane/python-sdk).
4. **Three byte-for-byte rules** matter when wiring auth (and most opaque `invalid_token` errors trace to one of them):
   - **Issuer URLs**: the AS-side `AUTHPLANE_SERVER_ISSUER` and the SDK-side `AUTHPLANE_ISSUER` must resolve to the same hostname.
   - **Resource URI**: what you register at the AS, what you pass to the SDK, and what the MCP client reaches must match — scheme, host, port, path — exactly.
   - **Scope name**: the same string appears in four places (SDK `scopes:` array, `POST /admin/resources`, `POST /admin/clients`, `POST /oauth/token`). Pick once, use everywhere.

> **Issuer decision — get this right before anything else.** `AUTHPLANE_SERVER_ISSUER` is what the **AS announces** (the `iss` it stamps and the host in its discovery doc); `AUTHPLANE_ISSUER` is where the **SDK discovers** metadata and fetches JWKS. They must point at the *same* host, and which host depends on topology:
> - **MCP server in the same Docker network as the AS?** → `http://authserver:9000` (the compose service name) on **both**.
> - **MCP server on the host, another machine, or public?** → `http://localhost:9000` (or your real public hostname) on **both**.
>
> Get this wrong and every call fails `401` with an opaque `invalid_token` even though the token is valid — the SDK fetched metadata from one host while the JWT's `iss` says another. This is the silent trap behind most "I did everything right and it still 401s" reports.

## Authoritative references — consult before generating any example

When generating ANY `docker run`, `curl`, admin-API call, env-var list, YAML config, or example code, consult the relevant file:

| File | What it covers | Generated from |
|---|---|---|
| [`docs/reference/cli.md`](docs/reference/cli.md) | Every `authserver` subcommand, every flag, value-format quirks (e.g. `--scopes 'name\|upstream\|description'` double-pipe for Mint) | `cmd/authserver/*.go` cobra tree |
| [`docs/reference/http-api.md`](docs/reference/http-api.md) | Every public + admin HTTP endpoint, request/response DTO, auth model | `api/{public,admin}/**` + DTO struct tags |
| [`docs/reference/env-vars.md`](docs/reference/env-vars.md) | Every `AUTHPLANE_*` env var, default, when required | `internal/config/loader.go` |
| [`docs/reference/configuration.md`](docs/reference/configuration.md) | Every YAML config key, type, default | `internal/config/config.go` |

If a flag, env var, or field a user needs isn't in the generated reference, **the docs are wrong before the example is**. File an issue against authserver; don't make one up.

## SDK pin per stack

Tell the user to install one of these, **at the exact version shown**. Each adapter is published on the public registry (PyPI / npm / `proxy.golang.org`). The versions below are CI-enforced against those registries by the `sdk-pins` gate: they can't silently drift from the published release, the same way the four generated references can't drift from source. If a newer version ships, the gate fails until the table *and* every example manifest are reconciled — don't infer a version from training data.

| Stack | Install (pinned) | Import / call |
|---|---|---|
| Python · FastMCP | `pip install authplane-fastmcp==0.2.0` | `from authplane_fastmcp import authplane_auth` |
| Python · official MCP Python SDK | `pip install authplane-mcp==0.2.0` | `from authplane_mcp import ...` |
| Python · any other framework (FastAPI, Starlette, raw ASGI) | `pip install authplane-sdk==0.2.0` | `from authplane import AuthplaneResource` |
| TypeScript · Express + `@modelcontextprotocol/sdk` | `npm i @authplane/mcp@0.2.0` | `import { authplaneMcpAuth } from "@authplane/mcp"` |
| TypeScript · FastMCP | `npm i @authplane/fastmcp@0.2.0` | `import { authplaneAuth } from "@authplane/fastmcp"` |
| TypeScript · any other framework | `npm i @authplane/sdk@0.2.0` | `import { AuthplaneResource } from "@authplane/sdk"` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AuthPlane/authserver](https://github.com/AuthPlane/authserver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
