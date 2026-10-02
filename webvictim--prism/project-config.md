---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & test commands

```bash
make                         # build bin/prism for the host
make prism-linux-amd64       # cross-compile for Linux amd64
make prism-linux-arm64       # cross-compile for Linux arm64
make prism-windows-amd64     # cross-compile bin/prism-windows-amd64.exe
make clean                   # remove bin/
go test ./...                # run all Go tests
go vet ./...                 # vet host target
GOOS=windows GOARCH=amd64 go vet ./...   # vet Windows target too
```

There is no linter wired up beyond `go vet`. End-to-end testing means
running `prism up && prism test && prism down` against a real cluster.
`prism test` exercises both backends; `prism test anthropic` and
`prism test openai` scope it to one tunnel.

## Architecture: direct cluster app tunnels

Prism tunnels traffic to two cluster-wide Teleport apps (`anthropic` and
`openai`) via `tsh proxy app` (interactive login) or `tbot` (Machine ID).
A local HTTP router on 127.0.0.1:7331 dispatches by path and applies
Bedrock-compatibility scrubbing to Anthropic requests.

```
Client → 127.0.0.1:7331 (local HTTP router + Bedrock scrubbing)
  /v1/chat/completions          → chatcompat shim → /v1/responses → openai tunnel
  /v1/responses, /v1/models, /v1/embeddings                      → openai tunnel
  /v1/messages, everything else                                  → anthropic tunnel
```

**tsh mode**: Two `tsh proxy app` subprocesses (anthropic + openai).
**tbot mode** (default when configured): One `tbot start` process with
two `application-tunnel` services in a generated tbot.yaml.

There are no beams, no embedded binaries, no in-beam proxy, and no
rotation. The cluster-wide apps are permanent and don't expire.

## Where the pieces live

```
cmd/prism/             local CLI (up, down, claude, codex, exec, daemon, etc.)
  daemon.go            starts tunnel services + router; branches tsh/tbot
  up.go                resolves identity, app login, picks ports, launches daemon
  claude.go            shared runToolWithPrism() for claude/codex/pi/exec
  pi.go                `prism pi` launcher + ~/.pi/agent/models.json setup
  usage_cmd.go         `prism usage` subcommand (reads usage.jsonl)
  launchd.go           macOS LaunchAgent management (darwin only)
  systemd.go           systemd user service management (linux only)
  service_stub.go      no-op stubs for non-linux/non-darwin platforms
internal/router/       local HTTP router: path dispatch
  router.go            mux, proxy setup, path canonicalisation, request logging
  capture.go           usage-capture middleware (parsing lives in internal/capture)
internal/capture/      shared response capture: token usage, status/size;
                       used by both the router and the MITM proxy
internal/scrub/        shared request scrubbing (Bedrock + OpenAI compat);
                       used by both the router and the MITM proxy
internal/proxyerr/     shared ReverseProxy ErrorHandler classification
                       (client cancel vs real upstream failure);
                       used by both the router and the MITM proxy
internal/chatcompat/   /v1/chat/completions → Responses API shim
  chatcompat.go        handler + adaptive unsupported-parameter retry
  translate.go         request/reply body translation
  stream.go            SSE event translation
internal/mitm/         forward-proxy MITM for Claude Code Remote Control compat
  ca.go                CA generation/persistence, leaf cert issuance
  proxy.go             CONNECT handler: intercept anthropic, blind-tunnel rest
internal/logfile/      date-rotating log writer with compression
internal/tunnel/       subprocess supervisor (tsh proxy app or tbot) + health loop
internal/tbot/         tbot config rendering, sidecar, bootstrap/configure, diag probing
internal/identity/     polls tsh status, fires OnExpired/OnRecovered callbacks
internal/state/        ~/.config/prism/state.json persistence
internal/config/       ~/.config/prism/config.json (proxy, identity, tbot.dir, claude_forward_proxy_mode)
internal/usage/        token usage tracking (JSONL writer, reader, aggregation)
internal/tshwrap/      thin wrappers around tsh apps/status commands
```

## State file

`~/.config/prism/state.json` stores the daemon PID and port assignments.
Much simpler than before — no beam IDs, no certificates, no bearer tokens.

## Listener ports

A running prism in tbot mode owns four 127.0.0.1 listeners:

- **Router** (default 7331): user-facing HTTP. Path-dispatches to the
  tunnels and serves `/_prism/health`.
- **Anthropic tunnel** (~7333): internal, fronted by tsh/tbot.
- **OpenAI tunnel** (~7334): internal, fronted by tsh/tbot.
- **tbot diag** (~7332): tbot's `--diag-addr`; serves `/livez` and
  `/readyz/<service>`. Only present in tbot mode.

In tsh mode there's no diag port, so three listeners total. There is no
separate control/health port — `/_prism/health` is hung off the router.

## Identity backends

- **tsh** (default): uses the user's interactive `tsh login`. Subject to
  12-24h SSO expiry. The identity watcher detects expiry and restarts the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [webvictim/prism](https://github.com/webvictim/prism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
