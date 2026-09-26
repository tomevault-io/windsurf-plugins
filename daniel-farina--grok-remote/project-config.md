---
trigger: always_on
description: `grok agent` runs the Grok agent without the interactive TUI. It's the entry point used by editor integrations, headless automation, and shared-process setups. There are four subcommands plus a handful of options that apply to all of them.
---

# `grok agent` reference

`grok agent` runs the Grok agent without the interactive TUI. It's the entry point used by editor integrations, headless automation, and shared-process setups. There are four subcommands plus a handful of options that apply to all of them.

Captured from `grok agent --help` against:

```
grok 0.1.212 (b7b8204a484)
```

## Top-level shape

```
grok agent [OPTIONS] [COMMAND]

Commands:
  stdio     Run the agent over stdio
  headless  Run the agent headlessly over the Grok WebSocket relay
  serve     Run the agent as a WebSocket server
  leader    Run as the shared leader process for other clients
  help      Print help
```

## Options that apply to every subcommand

| Flag | Env | What it does |
|---|---|---|
| `--reauth` (alias `----reauthenticate`) | | Run authentication before starting the agent. Useful when the cached login is stale. |
| `-m, --model <MODEL>` | | Model ID to use (e.g. `grok-build`, `grok-4-fast`). |
| `--reasoning-effort <EFFORT>` | | Reasoning effort for reasoning models. Valid: `none`, `minimal`, `low`, `medium`, `high`, `xhigh`. |
| `--always-approve` | | Auto-approve all tool executions. Equivalent of headless yolo mode. |
| `--agent-profile <PATH>` | `GROK_AGENT` | Path to an agent profile file. See "Agent profiles" below. |
| `--leader` | | Connect to a shared leader process instead of starting a new agent. Lets multiple clients share one backend. Default comes from `[cli] use_leader` in `config.toml`. |
| `--no-leader` | | Start a fresh agent even when config enables leader mode. |
| `--grok-ws-origin <URL>` | | Override the WebSocket origin used by `headless`, `leader`, and `serve`. Internal. |
| `--grok-ws-url <URL>` | | Override the WebSocket URL. Internal. |
| `--cli-chat-proxy-base-url <URL>` | | Override the CLI chat proxy base URL. (This is what grok-bench's proxy intercepts when you set `base_url` in `config.toml`.) |
| `--xai-api-base-url <URL>` | | Override the public xAI API base URL. |
| `-h, --help` | | Print help. |

## `grok agent stdio`

Run the agent over standard input/output. Each line on stdin is fed to the agent; each line on stdout is its response. Designed to be embedded by another process via a child-process pipe (editor integrations, scripting, ACP/MCP clients).

```
Usage: grok agent stdio

Options:
  -h, --help  Print help
```

No subcommand-specific options. Inherits all top-level options listed above.

**Example:**
```sh
# Pipe a prompt in and read the response out.
echo '{"role":"user","content":"hello"}' | grok agent stdio --model grok-build
```

## `grok agent headless`

Run the agent headlessly over the Grok WebSocket relay. This is the mode that backs cloud sessions and any remote-relay-based client.

```
Usage: grok agent headless [OPTIONS]

Options:
      --grok-ws-origin <GROK_WS_ORIGIN>
      --grok-ws-url <GROK_WS_URL>
  -h, --help                             Print help
```

The two `--grok-ws-*` overrides duplicate the top-level ones; useful when you want to make the override explicit at the subcommand level.

**Example:**
```sh
grok agent headless --model grok-build --reasoning-effort medium
```

## `grok agent serve`

Run the agent as a WebSocket server other clients connect to over the network. Lets you run the agent on one machine and use it from another, or share one backend between many clients.

```
Usage: grok agent serve [OPTIONS]

Options:
      --bind <BIND>
          Address for the server to listen on [default: 127.0.0.1:2419]
      --secret <SECRET>
          Secret token for client authentication (auto-generated if not provided)
          [env: GROK_AGENT_SECRET=]
      --remote <REMOTE>
          Remote agent URL for proxy mode
      --grok-ws-origin <GROK_WS_ORIGIN>
      --grok-ws-url <GROK_WS_URL>
  -h, --help
          Print help
```

Boot output prints something like:

```
Grok agent server starting...
Agent server listening on ws://127.0.0.1:2419
Clients should connect with: --remote ws://127.0.0.1:2419/ws --secret <token>
```

Authentication: clients must present the secret token. If not passed via `--secret`, one is auto-generated and printed on startup; or set `GROK_AGENT_SECRET` in env.

**Example:**
```sh
# Start the server on the local network with a known token.
GROK_AGENT_SECRET=hunter2 grok agent serve --bind 0.0.0.0:2419

# Connect another grok client to it:
grok agent stdio --remote ws://server:2419/ws --secret hunter2
```

**Proxy mode** (`--remote`): forward all traffic to another running agent server instead of running the agent locally. Useful for fan-out or for putting a closer-to-user listener in front of a slower remote.

## `grok agent leader`

Run as the shared leader process other agent clients connect to via `--leader`. The leader holds session state, MCP connections, skill caches, and the model conversation; many clients can ride the same leader and share that state. Default enabled by `[cli] use_leader = true` in `config.toml`.

```
Usage: grok agent leader [OPTIONS]

Options:
      --no-exit-on-disconnect            Keep the leader running after the last client disconnects

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daniel-farina/grok-remote](https://github.com/daniel-farina/grok-remote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
