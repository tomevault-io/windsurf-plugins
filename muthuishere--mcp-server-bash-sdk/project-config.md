---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Model Context Protocol server SDK in pure Bash, implementing MCP revision
**2026-07-28** over both standard transports. Three layers, split by file:

- `mcpserver_core.sh` — the protocol layer. JSON-RPC framing, method dispatch, per-request
  version negotiation, result envelopes, error codes. It defines functions only; sourcing
  it starts nothing. `run_mcp_server` is the stdio entry point.
- `mcpserver_http.sh` — the Streamable HTTP binding. Parses HTTP, enforces the header↔body
  contract, maps errors onto status codes, then calls **the same `process_request`**. It
  contains no protocol logic, which is what keeps the two transports from drifting.
- `moviemcpserver.sh`, `examples/gitserver.sh` — *implementations*. Set the config
  variables, source the core (and optionally the HTTP layer), define `tool_*` functions,
  dispatch on `--http`.

Users copy an implementation, not the core. `examples/README.md` is the walkthrough.

## Commands

```bash
./test_mcpserver_core.sh                        # 31 unit tests, one per spec scenario
./test_mcpserver_core.sh test_discover_shape    # a single test by function name
./test_conformance.sh                           # validate live output against the official schema
./test_http_transport.sh                        # 19 HTTP tests (drives a real server over curl)
./test_http_transport.sh test_bad_origin_is_403 # a single HTTP test
./docs/spikes/spike-01-stdio-viability.sh       # re-measure the stdio numbers
./docs/spikes/spike-02-http-listener.sh         # re-check listener/portability assumptions
./scripts/test-linux.sh all                     # run both suites on Debian, Alpine, Ubuntu (Docker)

# one-shot smoke test (note: every request MUST carry _meta protocolVersion)
echo '{"jsonrpc":"2.0","id":1,"method":"server/discover","params":{"_meta":{"io.modelcontextprotocol/protocolVersion":"2026-07-28","io.modelcontextprotocol/clientCapabilities":{}}}}' | ./moviemcpserver.sh
```

`jq` is the only hard dependency (plus `bc`, used by `tool_book_ticket`). `test_conformance.sh`
additionally needs Python with `jsonschema` and **skips cleanly** without it — point it at an
interpreter that has it with `MCP_CONFORMANCE_PYTHON=/path/to/python ./test_conformance.sh`.
No build step, no package manager, no linter.

## The contract between the two layers

Tool dispatch is by **shell function name**, not a registry: `handle_tools_call` looks up
`tool_<name>` with `type` and calls it with the arguments JSON as `$1`.

- A tool exists at runtime iff a `tool_<name>` function is in scope. `assets/*_tools.json`
  only controls what clients are *told* exists — the two can silently drift.
- Names are validated against `^[a-zA-Z0-9_]+$` before dispatch, so a call can never
  resolve to an arbitrary shell function.
- Success: `echo` the payload, `return 0`. Failure: `echo` the reason, `return 1` → a
  **successful** JSON-RPC response carrying `isError: true`, not a protocol error. This is
  deliberate ([ADR-0005](docs/adr/0005-tool-errors-are-results.md)): the model must see
  tool failures to self-correct. Only *finding* the tool failing is a protocol error.
- Tool output is stringified with `jq --arg`, so newlines survive as `\n` inside the JSON
  string. Content is always a single `content[0].type == "text"` block; structured content
  is not supported.

Configuration variables must be set **before** `source mcpserver_core.sh` — the core reads
them at source time via `${VAR:-default}`:

| Variable | Purpose | Core's default (does not exist in repo) |
| --- | --- | --- |
| `MCP_CONFIG_FILE` | the `server/discover` body + `serverInfo` | `assets/mcpserverconfig.json` |
| `MCP_TOOLS_LIST_FILE` | `{"tools":[…]}` | `assets/tools_list.json` |
| `MCP_LOG_FILE` | append-only log | `mcpserver.log` |
| `MCP_DEFAULT_TTL_MS` / `MCP_DEFAULT_CACHE_SCOPE` | cache hints when the config omits them | `60000` / `public` |

Unlike the old core, config files are no longer echoed back verbatim — `handle_discover`
and `handle_tools_list` *decorate* them with `resultType`, `ttlMs` and `cacheScope`, and
`create_response` injects `_meta` serverInfo into every result. A missing or malformed
config file yields `-32603` rather than a malformed result.

Logging is file-only by design: stdout is the protocol channel and any stray `echo` in a
tool corrupts the stream. Use `log LEVEL "msg"`; set `MCP_LOG_STDERR=1` to mirror to stderr.

## Protocol shape (2026-07-28)

MCP is now **stateless** — this is the thing to internalise before editing the core:

- **No `initialize`, no `notifications/initialized`.** They are gone from the protocol, not
  just unimplemented. Every request carries
  `params._meta["io.modelcontextprotocol/protocolVersion"]` and `…/clientCapabilities`.
- `process_request` gates on that version *once*, before the method `case`, answering a
  mismatch with `-32022` and `data: {requested, supported[]}`. **`server/discover` is
  exempt** — it is how a client learns which versions exist, so gating it would deadlock
  negotiation.
- Every result carries `resultType: "complete"` and `_meta` serverInfo. List results carry

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [muthuishere/mcp-server-bash-sdk](https://github.com/muthuishere/mcp-server-bash-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
