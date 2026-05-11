---
trigger: always_on
description: Go reverse proxy that exposes Anthropic, Gemini, and OpenAI-compatible APIs behind one local endpoint. Vekil can run in zero-config mode against GitHub Copilot, or use explicit provider routing to send selected models to configured upstreams such as Azure OpenAI and OpenAI Codex. The public API surface stays the same while provider ownership of models is configured behind the proxy.
---

# Contributor Guide

## Project

Go reverse proxy that exposes Anthropic, Gemini, and OpenAI-compatible APIs behind one local endpoint. Vekil can run in zero-config mode against GitHub Copilot, or use explicit provider routing to send selected models to configured upstreams such as Azure OpenAI and OpenAI Codex. The public API surface stays the same while provider ownership of models is configured behind the proxy.

## Build & Test

```bash
make build          # build binary
make test           # run all tests (go test ./... -count=1)
make vet            # go vet ./...
make lint           # runs vet
make build-app      # macOS menubar .app bundle
make docker-build   # docker image
```

Run specific tests: `go test ./proxy/ -run TestHandle -v`
Run websocket benchmark: `go test ./proxy/ -run '^$' -bench 'BenchmarkResponsesWebSocketRequestBuild' -benchmem -count=1`
Run transport benchmark: `go test ./proxy/ -run '^$' -bench 'BenchmarkResponsesTransport' -benchmem -count=1`
Run session benchmark: `go test ./proxy/ -run '^$' -bench 'BenchmarkResponsesSession' -benchmem -count=1`

## Documentation

Documentation is intentionally split under `docs/` into small, single-purpose files:

| File | Scope |
|------|-------|
| `README.md` | Short landing page only |
| `docs/README.md` | Documentation index and doc map |
| `docs/getting-started.md` | install, run, first auth, deployment entry points |
| `docs/configuration.md` | flags, env vars, provider routing, websocket tuning |
| `docs/clients.md` | copy-paste client examples |
| `docs/api.md` | endpoint behavior and compatibility notes |
| `docs/architecture.md` | package boundaries and design notes |
| `docs/menubar.md` | macOS/Linux tray app usage |
| `docs/development.md` | build, test, benchmark, CI |

Documentation update rules:

- Keep `README.md` concise; put detail into the focused file under `docs/`.
- When behavior changes, update the narrowest relevant doc instead of appending to the root README.
- Prefer linking between docs rather than duplicating long sections.
- Separate provider-agnostic behavior from provider-specific auth and routing details when possible.

## Architecture

| Package | Purpose |
|---------|---------|
| `main.go` | CLI entry, flags (including JSON/YAML providers config), signal handling |
| `auth/` | GitHub OAuth device code flow and Copilot token caching/refresh (`sync.RWMutex` + double-check) |
| `proxy/chat_handlers.go` | Anthropic and OpenAI chat handlers, including forced-streaming aggregation for tool calls |
| `proxy/responses_handler.go` | OpenAI Responses passthrough plus Codex compatibility endpoints (`/v1/responses/compact`, `/v1/memories/trace_summarize`) |
| `proxy/handler.go` | Shared proxy plumbing: health/ready/models handlers, request-body decoding, provider-aware model catalogs, provider headers, caches |
| `proxy/providers.go` | JSON/YAML provider config loading, model ownership, Azure metadata overlay, endpoint allowlists, and routing state |
| `proxy/upstream_http.go` | Provider selection, public→upstream model rewriting, and provider-specific upstream HTTP dispatch |
| `proxy/openai_codex_auth.go` | OpenAI Codex CLI auth.json loading, refresh, and request credentials |
| `proxy/gemini_handler.go` | Gemini-native HTTP handlers and countTokens probe flow |
| `proxy/gemini.go` | Gemini↔OpenAI request/response translation and validation |
| `proxy/gemini_streaming.go` | OpenAI SSE → Gemini SSE translation |
| `proxy/translator.go` | Bidirectional Anthropic↔OpenAI request/response translation |
| `proxy/streaming.go` | SSE stream translation (OpenAI→Anthropic events) and aggregation |
| `proxy/retry.go` | Exponential backoff on 429/502/503/504 |
| `models/` | Data-only structs for Anthropic and OpenAI API types (no logic) |
| `logger/` | Structured JSON logger to stderr |
| `server/` | HTTP server lifecycle (`Start`/`Stop`/`IsRunning`) |
| `cmd/menubar/` | tray app binary |

## Key Design Decisions

- **No frameworks**: Pure `net/http` with Go 1.22+ `ServeMux` method routing. Do not add web frameworks.
- **Vekil is a multi-provider proxy**: zero-config startup currently targets GitHub Copilot, but explicit JSON/YAML provider configs can extend or replace that default behind the same public API surface.
- **Public model IDs are global across providers**: Model ownership is explicit and startup must fail on collisions rather than silently shadowing one provider with another.
- **Forced streaming for reliable parallel tool calls**: Non-streaming requests with tools may be force-streamed upstream then aggregated back before returning to the client. This behavior started as an upstream compatibility workaround and still applies to provider-backed OpenAI chat handling.
- **Gemini is a translation layer**: Gemini endpoints are implemented like Anthropic, not as zero-copy passthrough. Keep Gemini-specific protocol logic in `proxy/gemini*.go`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sozercan/vekil](https://github.com/sozercan/vekil) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
