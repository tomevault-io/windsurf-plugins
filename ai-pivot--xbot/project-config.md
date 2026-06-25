---
trigger: always_on
description: > Go AI Agent framework with message bus + plugin architecture. Supports Feishu/QQ/CLI/Web channels, tool calling, pluggable memory, skills, subagents, MCP integration.
---

# xbot

> Go AI Agent framework with message bus + plugin architecture. Supports Feishu/QQ/CLI/Web channels, tool calling, pluggable memory, skills, subagents, MCP integration.

## Quick Reference

- Entry points: `cmd/xbot-cli/` (CLI), `cmd/runner/` (remote sandbox), `cmd/xbot/` (server)
- Build: `go build ./...` | Test: `go test ./...` | Lint: `golangci-lint run ./...`
- Config: `~/.xbot/config.json`, env var overrides
- Subscriptions: `~/.xbot/config.json` (CLI) or DB `user_llm_subscriptions` (Server) — the single source of truth for LLM config
- Pre-commit: gofmt → golangci-lint → go build → go test

## Knowledge Files

- `docs/agent/architecture.md` — package map, message flow, pipeline, Transport (Call+Close)/Backend/DirectBackend/Lifecycle separation, key interfaces, concurrency, TokenTracker, CompressPipeline, PersistenceBridge
- `docs/agent/agent.md` — agent loop, middleware, SubAgent, context management, masking, dynamic context, reminder
- `docs/agent/llm.md` — LLM clients, streaming pitfalls, retry behavior, model tiers (vanguard/balance/swift)
- `docs/agent/subscription.md` — **subscription system 完整文档**: LLMFactory cache、GetLLM/GetLLMForChat/GetLLMForModel 解析链、max_context 优先级、所有切换场景（per-session/全局/settings/启动恢复）、会话隔离规则、Invalidate 速查表、TUI↔Backend 数据同步
- `docs/agent/tools.md` — built-in tools: Shell, Read, Edit, Glob, Grep, Cd, Fetch, WebSearch, Cron, SubAgent, CreateChat, SendMessage, Worktree, config, tui_control, TodoWrite, context_edit, AskUser, DownloadFile, ChatHistory, ManageTools, Skill, EventTrigger, TaskManager, hooks system (agent/hooks/), sandbox types, ChannelPluginTransport (stdio channel plugin transport)
- `docs/agent/settings.md` — settings system: single registry (agent/setting_runtime.go), cli_settings.go, UpdatePerModelConfig, subscription-scoped vs user-scoped, runtime apply chain
- `docs/agent/conventions.md` — error handling, logging, testing, naming, build, **local/remote unification**
- `docs/agent/hooks.md` — hooks lifecycle events, handler types, configuration, gotchas
- `docs/agent/channel.md` — CLI (BubbleTea TUI), Feishu, Web, QQ adapters, asyncCh pattern, deterministic rendering, mouse support, settings panels
- `docs/agent/memory.md` — letta vs flat providers
- `docs/agent/conventions.md` — error handling, logging, testing, naming, build
- `docs/agent/plugin.md` — plugin system architecture, runtimes, integration, RPC bridge
- `docs/agent/worktree.md` — git worktree-based multi-agent workspace isolation, WorktreeRegistry, AutoDetectAndInit, peer discovery, path security

## Gotchas — MUST READ Before Any Code Change

### Concurrency
- **Never `defer` semaphore release inside a loop.** Deadlock when iterations exceed capacity. Release immediately after Generate completes.
- Non-blocking channel sends: always use `select` with `ctx.Done()` to prevent blocking on full channels during shutdown.
- **User-scoped semaphores must not be hardcoded to capacity 1 when one sender can own multiple independent chats/sessions (for example remote CLI windows authenticated as `admin`).** Size them from configured concurrency or key them by session, otherwise different windows will block each other and look like a leaked semaphore.
- **`SetMaxConcurrency` must clear `userSemaphores` cache.** The global semaphore is rebuilt with the new capacity, but `getUserSemaphore` caches per-user channels in a `sync.Map` via `LoadOrStore`. Without `Clear()`, users with custom LLM keep using the cached semaphore with the OLD capacity forever. Symptom: setting max_concurrency to 100 has no visible effect.
- **`cancelChildSessions` must only cancel sessions with matching `parentKey`.** The old code called `cancelCurrent()` on ALL interactive sessions inside the `Range` loop before checking `parentKey`. This killed sibling/peer background agents when any single agent was unloaded or panicked — all N peer agents die simultaneously at whatever time the first one finishes. The `parentKey` check must happen BEFORE `cancelCurrent()`.
- **SubAgents must never outlive their direct creator — ALL completion paths need `cancelChildSessions`.** Foreground SubAgents wrap their own `fgRunCtx` (with `bgSessionCtxKey`+`bgParentKey`) so children derive context from the creator, not a grandparent. Send path `runCtx` also has these markers. Every natural completion path (foreground, bg, send) calls `runCancel()` + `cancelChildSessions(key)`. Without the context wrapper, a foreground B creating bg C would have C's context chain skip B entirely → B finishes, C keeps running.

### Subscription & Settings
- **`agent/setting_runtime.go` is the SINGLE source of truth for runtime setting handlers.** Both CLI and server use it. Never create a second handler registry — it will silently diverge.
- **`backendSubscriptionManager` is the SINGLE SubscriptionManager implementation.** No more local/remote/config variants. All subscription operations go through Backend interface → Transport (local or remote).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ai-pivot/xbot](https://github.com/ai-pivot/xbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
