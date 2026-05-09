---
trigger: always_on
description: **go-appsec/toolbox (sectool)** is a CLI and MCP toolkit for human and LLM agent collaboration for application security testing.
---

## Project Overview

**go-appsec/toolbox (sectool)** is a CLI and MCP toolkit for human and LLM agent collaboration for application security testing.

Key characteristics:
- MCP-primary architecture: single API serves both agents and CLI
- CLI is a thin client over MCP for human interaction
- Global config at `~/.sectool/config.json` (auto-created)
- All output in markdown format for LLM consumption
- Pluggable backend architecture (native built-in proxy or Burp MCP for HTTP, Interactsh for OAST, Colly for crawling)

## Build Commands

```bash
make build          # Build to bin/sectool
make build-cross    # Cross-compile (linux/darwin, amd64/arm64)
make test           # Quick tests (-short flag)
make test-all       # Full tests with -race and coverage
make lint           # Run golangci-lint and go vet
```

## Architecture

```
CLI Command → MCP Client → MCP Server → Backends (Built-in Proxy or Burp MCP, OAST, Crawler)
MCP Agent  → MCP Server → Backends (Built-in Proxy or Burp MCP, OAST, Crawler)
```

### Core Files

- `sectool/main.go` - Entry point; routes `mcp` subcommand to server mode, else CLI command dispatch
- `sectool/config/config.go` - Config loading/saving, defaults, auto-creation
- `sectool/mcpclient/client.go` - MCP client wrapper for CLI usage
- `sectool/mcpclient/tools.go` - Typed methods for each MCP tool
- `sectool/mcpclient/types.go` - Client-specific option types (*Opts structs)
- `sectool/bundle/bundle.go` - Client-side bundle file operations for export

### Protocol

- `sectool/protocol/workflow.go` - Workflow mode constants shared between service and mcpclient
- `sectool/protocol/types.go` - Shared MCP response types (used by both service and mcpclient)

### Service Layer

- `sectool/service/server.go` - MCP server lifecycle and backend coordination
- `sectool/service/mcp_server.go` - MCP server setup, tool registration, workflow handling
- `sectool/service/mcp_proxy.go` - Proxy and flow_get tool handlers (poll, flow_get, cookie_jar, rules)
- `sectool/service/mcp_replay.go` - Replay tool handlers (send, request_send)
- `sectool/service/mcp_crawl.go` - Crawl tool handlers (create, seed, status, poll, sessions, stop)
- `sectool/service/mcp_oast.go` - OAST tool handlers (create, poll, get, list, delete)
- `sectool/service/mcp_encode.go` - Encode/decode tool handlers (url, base64, html)
- `sectool/service/mcp_hash.go` - Hash tool handler (md5, sha1, sha256, sha512, HMAC)
- `sectool/service/mcp_jwt.go` - JWT decode tool handler
- `sectool/service/mcp_diff.go` - Diff tool handler (structured flow comparison)
- `sectool/service/mcp_reflection.go` - Reflection tool handler (parameter reflection detection)
- `sectool/service/mcp_notes.go` - Notes tool handlers (save, list) and flow listing attachment
- `sectool/service/mcp_respond.go` - Proxy responder tool handlers (respond_add, respond_delete, respond_list); native backend only
- `sectool/service/flags.go` - MCP server flag parsing (`--port`, `--workflow`, `--config`, `--notes`)
- `sectool/service/backend.go` - HttpBackend, ResponderBackend, OastBackend, CrawlerBackend interfaces
- `sectool/service/backend_http_native.go` - Native built-in proxy implementation of HttpBackend
- `sectool/service/backend_http_native_respond.go` - Native backend implementation of ResponderBackend
- `sectool/service/backend_http_burp.go` - Burp MCP implementation of HttpBackend
- `sectool/service/backend_oast_interactsh.go` - Interactsh implementation of OastBackend
- `sectool/service/smtputil.go` - SMTP email header parsing utilities
- `sectool/service/backend_crawler_colly.go` - Colly-based crawler implementation
- `sectool/service/capture_filter.go` - BuildCaptureFilter: compiles proxy exclusion patterns from config
- `sectool/service/search.go` - RE2 pattern compilation with LLM-friendly double-escape correction; flow content matching
- `sectool/service/httputil.go` - HTTP request/response parsing utilities
- `sectool/service/jsonutil.go` - JSON field modification utilities
- `sectool/service/types.go` - Service-specific request and internal types

### Proxy Package

- `sectool/service/proxy/types.go` - Core types (Header, RawHTTP1Request/Response, H2RequestData/Response, HistoryEntry, Target, RuleApplier)
- `sectool/service/proxy/parser.go` - HTTP/1.1 tolerant parser with wire fidelity
- `sectool/service/proxy/validate.go` - Optional request validation
- `sectool/service/proxy/server.go` - TCP listener, accept loop, protocol detection
- `sectool/service/proxy/handler_http1.go` - Plain HTTP and HTTP-over-TLS handler
- `sectool/service/proxy/handler_connect.go` - CONNECT tunnel and TLS MITM with ALPN negotiation
- `sectool/service/proxy/handler_http2.go` - HTTP/2 stream handler with HPACK
- `sectool/service/proxy/handler_websocket.go` - WebSocket frame proxying
- `sectool/service/proxy/cert.go` - CA and per-hostname certificate management
- `sectool/service/proxy/history.go` - Thread-safe history storage
- `sectool/service/proxy/filter.go` - CaptureFilter type, SetCaptureFilter/ShouldCapture
- `sectool/service/proxy/interceptor.go` - ResponseInterceptor interface for serving canned responses in place of upstream requests
- `sectool/service/proxy/compression.go` - gzip/deflate utilities

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [go-appsec/toolbox](https://github.com/go-appsec/toolbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
