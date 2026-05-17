---
trigger: always_on
description: A Go proxy server that turns a GitHub Copilot subscription into OpenAI and Anthropic-compatible API endpoints. Allows tools like Claude Code, Cursor, and any OpenAI/Anthropic client to use Copilot as the backend LLM provider. Go rewrite of [ericc-ch/copilot-api](https://github.com/ericc-ch/copilot-api).
---

# CLAUDE.md — copilot-proxy-go

## Project Overview

A Go proxy server that turns a GitHub Copilot subscription into OpenAI and Anthropic-compatible API endpoints. Allows tools like Claude Code, Cursor, and any OpenAI/Anthropic client to use Copilot as the backend LLM provider. Go rewrite of [ericc-ch/copilot-api](https://github.com/ericc-ch/copilot-api).

## Build & Run

```bash
# Build
go build -o copilot-proxy-go .

# Authenticate (GitHub OAuth device-code flow)
./copilot-proxy-go auth

# Start server (default: 127.0.0.1:4141)
./copilot-proxy-go start

# Listen on all interfaces
./copilot-proxy-go start --host 0.0.0.0

# Start with Claude Code interactive setup
./copilot-proxy-go start --claude-code

# Start with Codex CLI interactive setup
./copilot-proxy-go start --codex

# Check Copilot usage quota
./copilot-proxy-go check-usage

# Debug info
./copilot-proxy-go debug [--json]
```

Go version: 1.25 (per go.mod)

## Testing

```bash
go test -v ./...
```

Note: Test files cover setup helpers. CI runs build + test.

## Project Structure

```
main.go                              # Entry point, cobra CLI commands (start/auth/check-usage/debug)
internal/
  api/
    config.go                        # API constants, headers, VS Code version fetcher
    errors.go                        # HTTP error types and JSON error responses
  auth/auth.go                       # GitHub OAuth device-code flow, token management, auto-refresh
  config/config.go                   # JSON config file (per-model settings, API keys, defaults)
  handler/
    messages.go                      # POST /v1/messages — core Anthropic-compatible handler (3-tier routing)
    messages_native.go               # Native Messages API backend
    messages_utils.go                # SSE helpers, model checks, vision detection, CLAUDE.md extraction
    chat_completions.go              # POST /chat/completions (OpenAI passthrough)
    responses.go                     # POST /responses (Responses API passthrough)
    translate_chat.go                # Anthropic <-> Chat Completions translation
    translate_chat_stream.go         # Streaming: Chat Completions -> Anthropic SSE
    translate_responses.go           # Anthropic <-> Responses API translation
    translate_responses_stream.go    # Streaming: Responses API -> Anthropic SSE
    responses_stream_sync.go         # Stream ID sync for Responses passthrough
    types_anthropic.go               # Anthropic request/response/stream types
    types_openai.go                  # OpenAI Chat Completions types
    types_responses.go               # OpenAI Responses API types
    quota.go                         # Compact/warmup detection, small model routing
    count_tokens.go                  # POST /v1/messages/count_tokens (estimation)
    models.go                        # GET /models
    health.go, token.go, usage.go    # Utility endpoints
    stats.go                         # GET /api/stats — aggregated metrics JSON endpoint
    dashboard.go, dashboard.html     # Embedded HTML dashboard (go:embed)
    embeddings.go                    # POST /embeddings passthrough
  logger/logger.go                   # Per-handler file logging with daily rotation (7-day retention)
  middleware/
    auth.go                          # API key auth (x-api-key / Bearer)
    ratelimit.go                     # Rate limiting (reject or wait mode)
    approval.go                      # Manual CLI approval per request
  server/server.go                   # chi router setup, all routes, middleware chain
  service/copilot.go                 # Copilot API proxy functions (all backend HTTP calls)
  shell/
    shell.go                         # Shell detection, export script generation
    clipboard.go                     # Cross-platform clipboard
  state/
    state.go                         # Thread-safe global state singleton (tokens, models, paths)
    metrics.go                       # In-memory metrics store (ring buffer, aggregates, session snapshots)
pages/index.html                     # Standalone usage dashboard
```

## Architecture

### Request Flow (Messages endpoint — most complex)

1. Parse Anthropic request → apply quota optimizations (compact/warmup → small model)
2. Detect subagent markers, merge tool result blocks
3. Update session snapshot (CLAUDE.md extraction, tools, thinking config)
4. Route to best backend based on model capabilities:
   - **Native Messages API** (`/v1/messages`) — passthrough with thinking/vision adjustments
   - **Responses API** (`/responses`) — translate Anthropic ↔ Responses format
   - **Chat Completions** (`/chat/completions`) — translate Anthropic ↔ Chat Completions format
5. Handle streaming (SSE event translation) or non-streaming (JSON translation)
6. Record request metrics (tokens, latency, backend, model) to `state.Metrics`

### Routes (chi router)

```
GET  /                              → Health
GET  /token                         → Token
GET  /usage                         → Usage

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tonghaoch/copilot-proxy-go](https://github.com/tonghaoch/copilot-proxy-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
