---
trigger: always_on
description: The agent API wraps exec mode in an HTTP server, letting AI agents create isolated WASM sandboxes, upload files, execute code, and retrieve structured output, all via REST.
---


# Agent API

The agent API wraps exec mode in an HTTP server, letting AI agents create isolated WASM sandboxes, upload files, execute code, and retrieve structured output, all via REST.

## Starting the Server

```sh
wasmrun agent [OPTIONS]
```

| Flag | Default | Description |
|------|---------|-------------|
| `-P, --port` | `8430` | Server port |
| `-t, --timeout` | `300` | Default session idle timeout (seconds) |
| `-m, --max-sessions` | `100` | Maximum concurrent sessions |
| `--max-memory` | `256` | Maximum linear memory per session (MB) |
| `--max-fuel` | `0` | Instruction budget per execution (`0` = unlimited) |
| `--max-output` | `10` | Captured stdout+stderr per execution (MB) |
| `--max-file-size` | `50` | Maximum size of any single file write (MB) |
| `--max-disk` | `100` | Maximum total disk usage per session (MB) |
| `--max-body` | `32` | Maximum accepted request body size (MB) |
| `--max-concurrent-exec` | `100` | Maximum executions in flight across all sessions |
| `--npm-registry` | `https://registry.npmjs.org` | npm registry base URL for dependency vendoring |
| `--allow-cors` | off | Enable wildcard CORS |
| `-v, --verbose` | off | Add a request-received line per request (a structured access log is always emitted; see [Observability](./usage/agent-observability.md)) |
| `--auth <PATH>` | off | Path to a TOML auth config; enables API-key auth & tenant isolation (omit = open) |
| `--hash-key <KEY>` | - | Print `sha256(KEY)` for the auth config and exit (does not start the server) |

For every size/count limit, `0` means **unlimited**. Memory, fuel, output, file-size, and disk caps are **per session** and can be overridden per session at creation (see [Sessions](./usage/agent-sessions.md)); body size and exec concurrency are **server-wide** ingress guards.

All endpoints are under `http://<host>:<port>/api/v1/`.

## Authentication

By default the server is **open**; any caller can create and access any session. Pass `--auth <path>` to require an API key on every request and isolate sessions per tenant. Without `--auth`, behavior is exactly as before (no header needed).

```sh
wasmrun agent --port 8430 --auth ./auth.toml
# banner shows:  Auth:  enabled (2 tenants)
```

### Config file

The auth config is a TOML file listing tenants. Keys are stored **hashed** (SHA-256, hex), never in plaintext:

```toml
[[tenants]]
id = "copilot"
key_sha256 = "9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08"

[[tenants]]
id = "ci"
key_sha256 = "60303ae22b998861bce3b28f33eec1be758a213c86c93c076dbe9f558c11c752"
```

Each `id` and `key_sha256` must be unique, and `key_sha256` must be 64 lowercase hex characters. Invalid or missing config **aborts startup**; the server never silently runs open when auth was requested. Restrict the file so other users can't read the hashes:

```sh
chmod 600 auth.toml
```

### Generating a key hash

Generate a high-entropy random key, then hash it for the config:

```sh
KEY=$(openssl rand -hex 32)
wasmrun agent --hash-key "$KEY"
# → 4b4090ccee1e713c3d411b96a4226b90bd0f0deb34e02d19475a951316fd04ee
```

Put the hash in `key_sha256`, hand the raw `$KEY` to that tenant, and keep the raw key out of the config.

### Making authenticated requests

Send the raw key as a Bearer token on every `/api/v1/*` request (including `/tools`):

```sh
curl -X POST http://localhost:8430/api/v1/sessions \
  -H "Authorization: Bearer $KEY"
```

A missing, malformed, or unknown key returns **401 Unauthorized**.

### Tenant isolation

Each session is owned by the tenant that created it. A tenant can only see and operate on its own sessions; any request targeting another tenant's session returns **404 Not Found**, identical to a nonexistent session so existence isn't leaked.

### Per-tenant limits and rate limits

Each tenant can carry its own resource ceiling and request budget, layered on top of the server defaults. Both are optional sub-tables under a `[[tenants]]` entry:

```toml
[[tenants]]
id = "ci"
key_sha256 = "60303ae22b998861bce3b28f33eec1be758a213c86c93c076dbe9f558c11c752"

  [tenants.limits]
  max_memory_mb = 128
  max_disk_mb = 50

  [tenants.rate]
  max_sessions = 10
  max_concurrent_exec = 4
  max_requests_per_min = 600
```

`[tenants.limits]` sets a per-tenant resource ceiling, with the same fields as a [per-session override](./usage/agent-sessions.md#per-session-limit-overrides) (`max_memory_mb`, `max_fuel`, `max_output_mb`, `max_file_size_mb`, `max_disk_mb`). Effective session limits compose in three layers: **server defaults → tenant baseline → per-session override clamped to the tenant baseline**. The tenant ceiling is a hard cap; a per-session override may only *tighten* a dimension, never raise it above the tenant's cap (a per-session "unlimited" `0` is pulled down to the tenant's finite ceiling).

`[tenants.rate]` throttles the tenant independently so one tenant cannot exhaust the shared server: `max_sessions`, `max_concurrent_exec`, `max_requests_per_min` (each `0` or omitted inherits the server-wide default). Over any of these limits returns **429 Too Many Requests**.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anistark/wasmrun](https://github.com/anistark/wasmrun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
