---
trigger: always_on
description: A network proxy tool for AI agents — a MITM proxy for AI.
---

# yorishiro-proxy

A network proxy tool for AI agents — a MITM proxy for AI.
Operates as an MCP (Model Context Protocol) server, providing traffic interception, recording, and replay capabilities for vulnerability assessment.

**Status**: OSS (Apache License 2.0) · Under active development

## Active Rewrite (RFC-001)

> **M36-M44 (Codec + Pipeline + Session) is cancelled.** Superseded by [RFC-001: Envelope + Layered Connection Model](docs/rfc/envelope.md) which fixes the HTTP bias in the Exchange/Codec abstractions.
> **Before working on any N1-N9 Issue, invoke the `/rfc001` skill** — it loads the spec, implementation guide, Linear state, and presents the implementation rules automatically.

- **Spec**: [`docs/rfc/envelope.md`](docs/rfc/envelope.md) (English) / [`docs/rfc/envelope-ja.md`](docs/rfc/envelope-ja.md) (Japanese)
- **Implementation strategy**: [`docs/rfc/envelope-implementation.md`](docs/rfc/envelope-implementation.md)
- **Branch**: `rewrite/rfc-001`
- **Active milestones**: N1-N9 (see Linear). M36-M44 are `[Cancelled]` in Linear.
- **Entry point**: `/rfc001` skill or `/rfc001 <Issue ID>`

## Architecture

### Principle: L7-first, L4-capable

1. **The default operation interface is a structured L7 view** — Prioritize AI agent token efficiency by representing communication as structured data: method, URL, headers, body, etc.
2. **Raw bytes recording, viewing, and modification must be possible for all protocols** — As a diagnostic tool, protocol-level anomaly detection and reproduction must be supported (pure transport-layer protocols like SOCKS5 apply to the tunneled protocol)
3. **L7 parsing is an overlay on top of raw bytes; the wire-observed raw bytes snapshot itself must not be destroyed or modified** — Recorded raw bytes always reflect the original wire data; modifications must always be treated as separate derived data (e.g., modified variant)

### Pipeline

```
TCP Listener (Layer 4)
  → Protocol Detection (peek bytes)
    → Protocol Handler (HTTP/S, HTTP/2, gRPC, WebSocket, Raw TCP)
      → Session Recording (L7 structured + L4 raw bytes)
        → MCP Tool (Intercept / Replay / Search)
```

### L7/L4 Support Status by Protocol

| Protocol | L7 Structured View | L4 raw bytes | Notes |
|----------|-------------------|--------------|-------|
| HTTP/1.x | YES | YES (parser built-in) | Independent engine in M32; net/http removed |
| HTTP/2 | YES | YES (frame codec) | Custom frame engine implemented in M26 |
| gRPC | YES | YES (via HTTP/2) | |
| gRPC-Web | YES | YES (via HTTP/1.x or HTTP/2) | M33; binary + base64 wire formats |
| WebSocket | YES | YES (per frame) | |
| Raw TCP | N/A | YES (byte stream) | |
| SOCKS5 | N/A | N/A (excluded as transport layer itself) | Apply raw bytes/L7 to protocol delegated after handshake/tunnel |

### Design Principles

- Accept connections at Layer 4 (TCP) and route to modular protocol handlers
- No external proxy libraries — built on standard library
- MCP-first: all operations are exposed as MCP tools

### MITM Implementation Principles

As a MITM proxy, yorishiro-proxy must faithfully represent wire-level reality. The following principles apply to all data path code (`internal/protocol/`, `internal/proxy/`, `internal/flow/`, `internal/plugin/`).

1. **Do not normalize what the wire did not normalize** — Header name casing, header order, duplicate headers with different casing, and whitespace must be preserved exactly as observed on the wire. If the wire sends `Set-Cookie: a=1` and `set-cookie: b=2`, they are two distinct headers with different names. Do not merge, canonicalize, or reorder.
2. **Each protocol has its own canonical form; do not unify across protocols** — HTTP/1.x headers are case-insensitive but preserve wire casing. HTTP/2 headers are lowercase by spec (RFC 9113). These are different realities and must be handled by protocol-specific code paths, not forced into a shared normalized representation.
3. **Prefer lossless representations over convenient ones** — Use ordered arrays (`[{name, value}, ...]`) over maps (`{name: [values]}`) for headers. Use protocol-native types (RawHeaders for HTTP/1.x, hpack.HeaderField for HTTP/2) over bridge types (gohttp.Header). Convenience helpers may be provided on top but must not be the storage format.
4. **`net/http` usage policy** — Data path code must not use `net/http` types for transport or data representation. Use internal types (RawRequest/RawResponse, hpack types). `net/http` is permitted only in the control plane: MCP server (`internal/mcp/`), CLI (`cmd/`), self-update (`internal/selfupdate/`), and status code constants (via `internal/protocol/httputil` shared package, see USK-522).

## Package Layout

```
cmd/yorishiro-proxy/       # Entry point
  main.go                  # CLI root: server/client/version/install/upgrade subcommands
  client.go                # CLI client subcommand (MCP client via Streamable HTTP)
  client_params.go         # Flag → JSON parameter conversion engine
  client_format.go         # Result formatting (JSON / table output)
  serverjson.go            # server.json multi-instance entry management (used for client auto-discovery)
internal/
  exchange/                # Protocol-agnostic message unit (Exchange, Direction, KeyValue)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [usk6666/yorishiro-proxy](https://github.com/usk6666/yorishiro-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
