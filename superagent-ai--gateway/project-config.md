---
trigger: always_on
description: Instructions for AI agents working in, or operating, this repository.
---

# AGENTS.md

Instructions for AI agents working in, or operating, this repository.

## What this is

`gateway` is a single-binary Rust HTTP gateway that lets coding agents use any
model provider through the protocol each client already speaks:

- **Claude Code** connects to an Anthropic Messages-compatible front door
  (`/anthropic/v1/messages`, alias `/v1/messages`)
- **Codex CLI** connects to an OpenAI Responses-compatible front door
  (`/openai/v1/responses`, alias `/v1/responses`)

Upstream providers speak either the Anthropic protocol or OpenAI Chat
Completions. The gateway translates bodies and streams in both directions,
routes by capability, and falls back conservatively: **never after output has
started streaming, never mid-tool-call**. That property is the reason this
project exists; do not weaken it.

## Build, test, run

```bash
cargo build                 # debug build
cargo test                  # all tests (unit + integration, no network needed)
cargo clippy --all-targets  # keep clippy clean
cargo run -- --config ./gateway.yaml   # run (default config path: ./gateway.yaml)
```

- Integration tests in `tests/integration.rs` spin up mock upstreams on
  ephemeral ports; they need no API keys or network.
- The server binds `127.0.0.1:4000` by default and also serves `[::1]`
  (Node clients resolve `localhost` to IPv6 first — do not remove this).
- `.env.local` / `.env` are auto-loaded at startup (existing env wins).

## Code map

```text
src/main.rs      CLI, env loading, bind safety (no public bind without tokens)
src/schema.rs    user-facing YAML format -> compiles into internal config
                 (provider presets, model quirk table, role/alias generation)
src/config.rs    internal routing config: models, routes, capabilities, glob map
src/classify.rs  request feature detection + route eligibility
src/http.rs      front doors, auth, route selection, attempt/fallback loop
src/translate.rs body translation (Anthropic Messages is the pivot format)
src/stream.rs    SSE parsing + stream translation state machines
src/error.rs     protocol-shaped errors; upstream errors forwarded verbatim
```

Request flow: front door → auth → resolve model name (exact → glob map →
unknown handling) → classify request features → filter eligible routes →
attempt loop (retry/fallback only before streaming) → translate body/stream
back to the client protocol.

## Configuration model

`gateway.yaml` has four sections; the file itself is the reference (it carries
commented examples of every option).

- `models:` is the single naming authority. `name: provider/model-id`, a list
  for a fallback chain, or a long form for capability overrides. The name is
  the model id clients use, verbatim, on both front doors.
- `clients:` maps traffic to models per client. Roles: `main`, `subagent`
  (Claude Code Task-tool requests, which pin `claude-opus-*` ids),
  `background` (`claude-haiku-*` ids: titles/summaries), `unknown` (anything
  unrecognized; default `main`, `reject` to 404).
- `providers:` only for endpoints the built-in presets don't cover. Presets
  (openrouter, openai, anthropic, moonshot, fireworks, together, groq,
  deepinfra, deepseek, mistral, xai, cerebras, ollama, azure) imply base URL
  and key env var.
- Known model families get capabilities from the quirk table in
  `src/schema.rs` (e.g. Kimi: `temperature`/`top_p` stripped because the API
  rejects them, reasoning preserved across tool turns, vision enabled).

Secrets never go in the YAML: provider keys come from env vars, any value can
be written as `${VAR}`, and `GATEWAY_BIND` / `GATEWAY_TOKEN` override the
listen address and auth token from the environment.

## Operating the gateway

### Start it

```bash
cargo run --release -- --config ./gateway.yaml
# or a prebuilt binary:
./target/release/gateway --config ./gateway.yaml
```

Verify: `curl http://127.0.0.1:4000/health` → `{"status":"ok",...}`.

### Connect Claude Code

Project-level `.claude/settings.json` (preferred — survives shell changes):

```json
{
  "model": "kimi",
  "env": {
    "CLAUDE_CODE_USE_BEDROCK": "0",
    "ANTHROPIC_BASE_URL": "http://127.0.0.1:4000/anthropic",
    "ANTHROPIC_AUTH_TOKEN": "local-dev-token",
    "CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY": "1"
  }
}
```

Notes learned the hard way:
- Use `127.0.0.1`, not `localhost` (IPv6 resolution), although the gateway
  now serves both loopbacks.
- A global `~/.claude/settings.json` forcing Bedrock/Vertex overrides
  everything and produces confusing 403s; neutralize it per project as above.
- Set `"model"` explicitly. Without it Claude Code requests its default
  model id (often `claude-opus-*`), which routes to the `subagent` role, not
  `main`.
- Bare model names from `gateway.yaml` are sent verbatim and always work
  (`claude --model <name>`). Discovery additionally lists each named model in
  the /model picker via a `claude-<name>` twin id (Claude Code refuses to
  list non-Claude-looking ids); both ids route identically. Discovery results
  are cached in `~/.claude/cache/gateway-models.json` — delete it if the
  picker shows stale entries.

### Connect Codex

User-level `~/.codex/config.toml` (project-local provider config is ignored
by Codex):

```toml
model = "kimi"
model_provider = "gateway"


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [superagent-ai/gateway](https://github.com/superagent-ai/gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
