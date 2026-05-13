---
trigger: always_on
description: API proxy that wraps the Claude CLI (`claude --print`) as a subprocess, exposing **Anthropic Messages API** and **OpenAI Chat Completions API** endpoints. Powered by the Claude Max subscription quota.
---

# Claude Code Proxy

API proxy that wraps the Claude CLI (`claude --print`) as a subprocess, exposing **Anthropic Messages API** and **OpenAI Chat Completions API** endpoints. Powered by the Claude Max subscription quota.

## Quick Reference

```bash
npm run build          # Compile TypeScript
npm start              # Run the proxy (port 4523)
claude-proxy           # Same, if globally linked via `npm link`
REQUIRE_AUTH=false claude-proxy   # Run without auth
```

## Architecture

Every HTTP request spawns a fresh CLI subprocess. The proxy is fully stateless.

```
HTTP Request
  -> Route Handler (routes/)
    -> Translate request to CLI args (translation/)
    -> Spawn `claude --print` subprocess (cli/)
    -> Parse NDJSON stdout stream (cli/stream-parser.ts)
    -> Translate CLI events to API response (translation/)
  -> HTTP Response (streaming SSE or JSON)
```

### Request Flow

```
Client SDK                    Proxy                           Claude CLI
    |                           |                                 |
    |-- POST /v1/messages ----->|                                 |
    |                           |-- translateAnthropicRequest() ->|
    |                           |-- buildArgs() ----------------->|
    |                           |-- spawnCli(args, prompt) ------>|
    |                           |       stdin: prompt             |
    |                           |       stdout: NDJSON events     |
    |                           |<-- stream_event (SSE) ---------|
    |<-- SSE chunks ------------|                                 |
    |                           |<-- result event --------------- |
    |<-- stream end ------------|                                 |
```

### OpenAI requests follow the same path

OpenAI format is normalized to Anthropic format first (`convertMessages()` in the route handler), then the shared Anthropic->CLI pipeline runs. Responses are translated back to OpenAI format.

## Directory Structure

```
src/
  index.ts                  # Entry point: config, CLI verification, HTTP server, shutdown
  config.ts                 # Environment variable loading (Config interface)

  cli/                      # Claude CLI subprocess management
    args-builder.ts         # Builds CLI argument arrays + extracts prompt for stdin
    stream-parser.ts        # NDJSON async generator (stdout -> CliEvent[])
    subprocess.ts           # spawn(), timeout, kill, event generator

  protocol/                 # Type definitions only (no logic)
    cli-types.ts            # CliEvent union: system|assistant|user|stream_event|rate_limit|result
    anthropic-types.ts      # Anthropic Messages API request/response/SSE types
    openai-types.ts         # OpenAI Chat Completions request/response/streaming types

  routes/                   # HTTP route handlers
    anthropic-messages.ts   # POST /v1/messages  (Anthropic format)
    openai-chat-completions.ts  # POST /v1/chat/completions  (OpenAI format)
    models.ts               # GET /v1/models
    health.ts               # GET /health

  server/                   # HTTP infrastructure
    app.ts                  # createServer(), route dispatch, CORS preflight
    middleware.ts           # Auth check, JSON body parsing, error responses, CORS headers

  tools/                    # MCP bridge for client-defined tool use
    tool-translator.ts      # Anthropic tool defs -> MCP server config
    mcp-bridge.ts           # Standalone MCP stdio server (child process)

  openclaw/                 # OpenClaw integration
    tool-map.ts             # Tool name mapping (exec→Bash, read→Read, etc.) + reverse map
    prompt-filter.ts        # Strip injected tooling sections from system prompts

  translation/              # Format conversion (the core logic)
    model-map.ts            # Model alias resolution, effort validation, model listing
    anthropic-to-cli.ts     # AnthropicMessagesRequest -> CliArgs (prompt + flags)
    cli-to-anthropic-stream.ts   # CliEvent async generator -> Anthropic SSE strings
    cli-to-anthropic.ts          # CliEvent async generator -> AnthropicMessagesResponse
    cli-to-openai-stream.ts      # CliEvent async generator -> OpenAI SSE strings
    cli-to-openai.ts             # CliEvent async generator -> OpenAIChatCompletionResponse

  util/
    errors.ts               # ApiError class + factory functions (badRequest, unauthorized, etc.)
    logger.ts               # JSON structured logger to stderr
```

## Key Modules — Where to Find Things

### "I need to change how requests are sent to the CLI"
- `src/cli/args-builder.ts` — `buildArgs()` constructs the flag array
- `src/cli/subprocess.ts` — `spawnCli()` manages the process lifecycle
- The prompt goes via **stdin** (not as a positional arg). The `buildArgs()` function returns `{ args, prompt }` separately.

### "I need to change how messages are converted to a prompt"
- `src/translation/anthropic-to-cli.ts` — `messagesToPrompt()` flattens the messages array into a single string. Multi-turn uses `<assistant_response>` and `<tool_result>` XML tags.

### "I need to add/change a model"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AntonioAEMartins/claude-code-proxy](https://github.com/AntonioAEMartins/claude-code-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
