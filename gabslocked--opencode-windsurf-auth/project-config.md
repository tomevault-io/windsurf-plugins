---
trigger: always_on
description: Guidance for AI agents working on this repository.
---

# AGENTS.md

Guidance for AI agents working on this repository.

## Overview

OpenCode plugin that exposes Windsurf/Codeium models as an OpenAI-compatible local API. Starts an HTTP server on `127.0.0.1:42100` providing `/v1/chat/completions` (streaming SSE) and `/v1/models`. Supports 90+ models including `swe-1.5`, `claude-4.5-opus-thinking`, `gpt-5.2`, `gemini-3.0-pro`, and more.

**Key insight**: Windsurf does NOT use REST APIs or cloud OAuth — it runs a local `language_server_macos` gRPC server. This plugin discovers credentials from the running process and translates REST ↔ gRPC.

## Build & Test

```bash
bun install           # install dependencies
bun run build         # compile TypeScript (tsconfig.build.json)
bun run typecheck     # type check only (tsconfig.json)
bun test              # run unit tests
bun test tests/unit   # unit tests only
```

**Runtime**: Requires **Bun** (uses `Bun.serve()`). Will not work under Node.js.

## Architecture

```
Client (OpenAI-format)
  → POST localhost:42100/v1/chat/completions
  → plugin.ts (Bun HTTP server)
    → auth.ts: discover CSRF, port, API key from running Windsurf
    → models.ts: resolve model name → protobuf enum value
    → discovery.ts: parse extension.js for protobuf field numbers
    → grpc-client.ts: encode protobuf, HTTP/2 POST to localhost:{port}
      → /exa.language_server_pb.LanguageServerService/RawGetChatMessage
  → Windsurf language server (local gRPC)
  → Windsurf cloud inference
  → Streaming protobuf response
  → grpc-client.ts: decode protobuf, yield text chunks
  → plugin.ts: wrap as SSE `data: {...}\n\n` chunks
  → Client receives OpenAI-format stream
```

## Module Structure

```
index.ts                    # Package entry, re-exports plugin
src/
├── plugin.ts               # Main: Bun HTTP server, OpenAI-compat endpoints,
│                           #   tool-call planning, streaming/non-streaming responses
├── constants.ts            # API endpoints, gRPC service paths, model IDs,
│                           #   keychain constants, rate limit config
└── plugin/
    ├── auth.ts             # Credential discovery: ps aux → CSRF, lsof → port,
    │                       #   sqlite3 state.vscdb → API key, process args → version
    ├── discovery.ts        # Dynamic protobuf field number discovery from extension.js
    │                       #   (survives Windsurf version updates)
    ├── grpc-client.ts      # HTTP/2 gRPC client with hand-rolled protobuf encoding/decoding
    │                       #   (no protobuf library), async generator for streaming
    ├── models.ts           # Model name → enum mappings (90+), variant system,
    │                       #   alias resolution, reverse enum→name mapping
    └── types.ts            # ModelEnum numeric values (extracted from extension.js),
                            #   ChatMessageSource enum, TypeScript interfaces

tests/
├── README.md               # Test documentation
├── unit/
│   └── variant.test.ts     # Model variant resolution tests
└── live/
    ├── request.ts          # Send test requests to running Windsurf
    ├── test-all-models.ts  # Test all model enum values
    ├── test-complete.ts    # Completion integration test
    ├── verify-plugin.ts    # Plugin verification
    ├── analyze.ts          # Analyze captured traffic
    ├── capture.sh          # tcpdump capture script (requires sudo)
    └── ngrep-capture.ts    # ngrep-based capture
    └── model-test-results.json  # Saved test results

docs/
├── WINDSURF_API_SPEC.md    # gRPC API specification (protobuf schemas)
└── REVERSE_ENGINEERING.md  # How credentials and protocol were discovered
```

## Credential Discovery (auth.ts)

| Credential | Source | Method |
|------------|--------|--------|
| CSRF token | Process args | `ps aux \| grep language_server_macos` → regex `--csrf_token ([a-f0-9-]+)` |
| gRPC port | Process sockets | Extract PID → `lsof -p PID -i -P -n \| grep LISTEN` → first port after `extension_server_port`. Fallback: `extension_server_port + 3` |
| API key | VSCode state DB | `sqlite3 ~/Library/Application Support/Windsurf/User/globalStorage/state.vscdb "SELECT value FROM ItemTable WHERE key = 'windsurfAuthStatus'"` → parse JSON → `.apiKey`. Fallback: `~/.codeium/config.json` |
| Version | Process args | regex `--windsurf_version ([^\s]+)` from process args. Fallback: `1.13.104` |

**Platform paths for state.vscdb**:
- macOS: `~/Library/Application Support/Windsurf/User/globalStorage/state.vscdb`
- Linux: `~/.config/Windsurf/User/globalStorage/state.vscdb`
- Windows: `%APPDATA%/Windsurf/User/globalStorage/state.vscdb`

## gRPC Protocol

### Endpoint
```
POST http://localhost:{port}/exa.language_server_pb.LanguageServerService/RawGetChatMessage
Headers:
  content-type: application/grpc
  te: trailers
  x-codeium-csrf-token: {csrf_token}
```

### Request Format (RawGetChatMessageRequest)
Hand-encoded protobuf (no library):
- Field 1: `Metadata` message (api_key, ide_name, ide_version, extension_version, session_id, locale)
- Field 2: `ChatMessage` repeated (message_id, source enum, timestamp, conversation_id, intent/text)
- Field 3: `system_prompt_override` string (if system message present)
- Field 4: `chat_model` varint (model enum value)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gabslocked/opencode-windsurf-auth](https://github.com/gabslocked/opencode-windsurf-auth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
