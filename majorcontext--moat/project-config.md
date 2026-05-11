---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Moat runs AI agents in isolated containers with credential injection and full observability. Key features:

- **Isolated Execution** - Each moat runs in its own container (Docker or Apple containers) with workspace mounting
- **Credential Injection** - Transparent auth header injection via TLS-intercepting proxy (agent never sees raw tokens)
- **Smart Image Selection** - Automatically selects container images based on `moat.yaml` runtime config
- **Full Observability** - Captures logs, network requests, and traces for every run
- **Declarative Config** - Configure agents via `moat.yaml` manifests
- **Multi-Runtime Support** - Automatically uses Apple containers (macOS 26+) or Docker

## Architecture

```
cmd/moat/           CLI entry point (Cobra commands)
internal/
  audit/             Tamper-proof audit logging with cryptographic verification
  claude/            Claude Code settings and Dockerfile generation
  cli/               Shared CLI helpers (environment parsing, mount helpers)
  codex/             Codex CLI settings and Dockerfile generation
  config/            moat.yaml parsing, mount string parsing
  container/         Container runtime abstraction (Docker and Apple containers)
  credential/        Secure credential storage (GitHub, Anthropic, AWS)
  daemon/            Proxy daemon lifecycle, Unix socket API, and run registration
  image/             Runtime-based image selection (node/python/go → base image)
  log/               Structured logging (slog wrapper)
  provider/          Provider registry and interfaces (CredentialProvider, AgentProvider)
  providers/         Provider implementations:
    aws/               AWS IAM role assumption and credential endpoint
    claude/            Claude Code CLI, grants, and config generation
    codex/             Codex CLI, grants, and config generation
    github/            GitHub token management and refresh
  run/               Run lifecycle management (create/start/stop/destroy)
  storage/           Per-run storage for logs, traces, network requests
  ui/                TTY-aware colored output and formatting helpers
```

### Key Flows

**Credential Injection:** `moat grant github` → token from gh CLI, env var, or PAT prompt → token stored encrypted → `moat run --grant github` → run registered with proxy daemon → container traffic routed through proxy → proxy resolves run by auth token → Authorization headers injected for matching hosts

**Image Selection:** `moat.yaml` `dependencies` field → `image.Resolve()` → node:X-slim / python:X-slim / golang:X / debian:bookworm-slim

**Observability:** Container stdout → `storage.LogWriter` → `~/.moat/runs/<id>/logs.jsonl`; Proxy requests → `storage.NetworkRequest` → `network.jsonl`

**Container Runtime Selection:** `container.NewRuntime()` auto-detects: Apple containers on macOS 26+ with Apple Silicon, otherwise Docker

**Audit Logging:** Console/network/credential events → `audit.Store.Append()` → hash-chained entries in SQLite → `moat audit <run-id>` displays chain with verification; `--export` creates portable proof bundle with attestations

**MCP Integration:** `moat.yaml` defines remote MCP servers → `.claude.json` generated with relay URLs → Claude Code connects to proxy relay → proxy injects credentials → request forwarded to real MCP server with SSE streaming support

### Proxy Daemon

The credential-injecting proxy runs as a shared daemon process that outlives the CLI. A single daemon serves all active runs.

- **Lifecycle:** Started automatically by `moat run` or manually via `moat proxy start`. Auto-shuts down after 5 minutes idle (no active runs).
- **Management API:** Unix socket at `~/.moat/proxy/daemon.sock`. The CLI registers/unregisters runs via this socket.
- **Per-run credential scoping:** Each run gets a cryptographic auth token (32 bytes from `crypto/rand`). The proxy looks up run-specific credentials, headers, network policy, and MCP config by token. Both Docker and Apple containers use token-based proxy auth (`HTTP_PROXY=http://moat:token@host:port`).
- **Responsibilities:** Credential injection, token refresh, MCP relay, hostname routing, and network request logging.
- **Lock file:** `~/.moat/proxy/daemon.lock` records PID, ports, and build commit.
- **Backwards compatibility:** The daemon API (`internal/daemon/api.go`) **must remain backwards-compatible across binary versions**. The daemon process outlives the CLI that spawned it, so older daemons serve newer CLIs and vice versa. Rules: additive-only fields, no removed/renamed fields, new endpoints must handle 404 gracefully. See the package doc in `api.go`.

See `github.com/majorcontext/gatekeeper/proxy` and `internal/daemon/` for implementation.

### MCP (Model Context Protocol) Support

Moat supports two types of MCP servers:

1. **Remote HTTP MCP servers** (top-level `mcp:` in moat.yaml) - External MCP servers accessed via HTTPS with credential injection through a proxy relay pattern
2. **Local process MCP servers** (under `claude.mcp:` or `codex.mcp:`) - MCP servers running as child processes inside the container


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [majorcontext/moat](https://github.com/majorcontext/moat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
