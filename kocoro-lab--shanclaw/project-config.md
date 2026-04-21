---
trigger: always_on
description: Go CLI tool (`shan`) — the runtime for Shannon AI agents. The primary production stack is **daemon + ShanClaw Desktop + Shannon Cloud**: the daemon connects to Cloud via WebSocket, receives channel messages, runs the agent loop locally with full tool access, and streams results back. ShanClaw also supports interactive TUI, one-shot CLI, MCP server mode, and local scheduled tasks.
---

# ShanClaw — Project Guide

## What This Is

Go CLI tool (`shan`) — the runtime for Shannon AI agents. The primary production stack is **daemon + ShanClaw Desktop + Shannon Cloud**: the daemon connects to Cloud via WebSocket, receives channel messages, runs the agent loop locally with full tool access, and streams results back. ShanClaw also supports interactive TUI, one-shot CLI, MCP server mode, and local scheduled tasks.

## Tech Stack

- **Go 1.25.7** — `go.mod` is source of truth
- **Cobra** — CLI framework
- **gorilla/websocket** — daemon WebSocket client (primary production path)
- **Bubbletea v1.3.10 + Bubbles v1.0.0** — TUI
- **modernc.org/sqlite** — pure-Go SQLite for session FTS5 search index
- **adhocore/gronx** — cron expression validation
- **chromedp** — browser automation (isolated Chrome profile)
- **mcp-go** — MCP client/server
- **adrg/frontmatter** — YAML frontmatter parsing for SKILL.md files

## Project Structure

```
cmd/
  root.go              # entry, one-shot mode, MCP serve
  daemon.go            # shan daemon start/stop/status
  schedule.go          # scheduled task management
  update.go            # self-update command

internal/
  daemon/              # PRIMARY PRODUCTION PATH
    server.go          # HTTP API server
    runner.go          # agent run orchestration, session routing, output format profiles
    client.go          # WebSocket client with reconnect, bounded concurrency
    router.go          # SessionCache, route locking
    approval.go        # interactive tool approval over WS
    types.go           # daemon request/response types, disconnect, approval messages
    events.go          # EventBus ring buffer for daemon/SSE subscribers
    session_cwd.go     # cloud-source scratch CWD allocator (ephemeral per-session tmp dir)
  agent/
    loop.go            # AgentLoop.Run() — core agentic loop
    tools.go           # Tool interface, ToolRegistry, filtering, schemas
    partition.go       # read-only batching, executeBatches
    spill.go           # large tool result spill-to-disk
    deferred.go        # deferred tool loading (tool_search)
    statecache.go      # state-aware tool result cache keyed by read/write state
    resultshape.go     # tree result shaping and stable change summaries
    microcompact.go    # Tier 2 semantic compaction for large native tool results
    delta.go           # DeltaProvider interface, TemporalDelta (date rollover)
    loopdetect.go      # stuck-loop detectors
    readtracker.go     # read-before-edit enforcement
    approval_cache.go  # per-turn approval caching
    normalize.go       # response normalization
    skill_discovery.go # Per-turn small-model skill matching (discoverRelevantSkills)
  agents/
    loader.go          # LoadAgent, ListAgents, ParseAgentMention
    api.go             # daemon-side agent CRUD
    validate.go        # agent validation and builtin commands
    embed.go           # EnsureBuiltins, MaterializeBuiltin, bundled agents
    builtin/           # Bundled agent definitions (explorer, reviewer)
  client/
    gateway.go         # GatewayClient: Complete, CompleteStream, ListTools
    sse.go             # SSE event parsing
    ollama.go          # Ollama provider via OpenAI-compatible chat/tool APIs
  config/
    config.go          # multi-level config loading and merge
    settings.go        # UI settings
    setup.go           # setup wizard
  cwdctx/
    cwdctx.go          # session-scoped CWD: context propagation, path resolution
  context/
    window.go          # token estimation, compaction shaping
    summarize.go       # two-phase conversation summary generation
    persist.go         # write-before-compact memory extraction
  session/
    store.go           # session JSON persistence
    manager.go         # session lifecycle, search, OnClose callbacks
    index.go           # SQLite FTS5 search index
    title.go           # session title generation
  prompt/
    builder.go         # static/stable/volatile prompt assembly, output profiles
  instructions/
    loader.go          # instructions, memory, custom commands
  tools/
    register.go        # local + MCP + gateway tool registration
    schedule.go        # schedule_create/list/update/remove tools
    session_search.go  # session_search tool
    mcp_tool.go        # MCPTool adapter
    server.go          # ServerTool adapter (gateway tools)
  skills/
    registry.go        # skill metadata
    loader.go          # skill loading
    validate.go        # skill name validation
  memory/
    types.go             # Wire schemas mirroring the Kocoro Cloud memory sidecar HTTP contract
    errclass.go          # ErrorClass enum + ClassifyHTTP (sub_code → class)
    config.go            # LoadConfig, ResolveAPIKey, ResolveEndpoint
    tenant.go            # sha256 fingerprint + tenant-switch detection
    audit.go             # AuditLogger interface (boolean-only key/endpoint state)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kocoro-lab/ShanClaw](https://github.com/Kocoro-lab/ShanClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
