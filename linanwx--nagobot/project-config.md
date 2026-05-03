---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

```bash
go build -o nagobot .          # Build
go test ./...                   # Run all tests
nagobot update                  # Self-update from GitHub Releases
```

Single package test: `go test ./provider -v -run TestSanitize`

## Architecture

nagobot is a Go-based AI bot framework. Messages flow through four layers:

```
Channel (I/O) → Dispatcher (routing) → Thread (execution) → Provider (LLM)
```

### Channel → Dispatcher (`channel/` → `cmd/dispatcher.go`)

Channels are pure I/O (Telegram, Discord, Feishu, Web, CLI, Cron). Each produces `channel.Message` structs. The Dispatcher routes messages to threads by computing a `sessionKey` (e.g., `"telegram:123456"`) and wrapping the message into a `WakeMessage` with Source, Sink, AgentName, and Vars.

The `/init` command is intercepted in the Dispatcher and executed directly via `initCmd.ParseFlags()` + `RunE()` — it does NOT go through the thread/LLM pipeline.

### Thread Manager (`thread/manager.go`)

Schedules up to 16 concurrent threads. `Manager.Wake(sessionKey, msg)` creates a thread if needed and enqueues the message. The Run() loop picks runnable threads and calls `RunOnce()`. Idle threads are GC'd after 3 hours.

### Thread Execution (`thread/run.go`, `thread/wake.go`, `thread/runner.go`)

`RunOnce()` dequeues a WakeMessage, merges consecutive same-source messages, builds the prompt, and runs the agentic loop (LLM call → tool execution → repeat). The `Runner` handles the iteration loop with hooks for streaming, message injection, and halt conditions.

Key: `resolveProvider()` calls `ProviderFactory.Create()` each time (not cached) so config changes from `/init` take effect immediately.

### WakeMessage Format (`thread/wake.go`)

Wake payloads use YAML frontmatter + markdown body with per-source sender:
- User messages (telegram/discord/cli/etc): `sender: user`
- System messages (child_completed/cron/sleep/heartbeat/etc): `sender: system`

Action hints for system sources explicitly tell the AI to include content in its response.

### Agent Templates (`agent/`)

Agents are markdown templates in `{workspace}/agents/{name}.md` with `{{PLACEHOLDER}}` syntax. Variables set via `agent.Set(key, value)` before `Build()`. Runtime vars (TOOLS, SKILLS, USER) are set per-turn in `thread/run.go`. `{{DATE}}` and `{{CALENDAR}}` are auto-resolved in `agent.Build()` at day-level granularity (no minutes/seconds).

**Important**: `{{WORKSPACE}}` is resolved in both `agent.Build()` and `use_skill` (`tools/skills.go`). Skills should use `{{WORKSPACE}}/bin/nagobot` for CLI calls.

### Provider Layer (`provider/`)

Each provider implements `Provider.Chat(ctx, *Request) (ChatResult, error)`. `ChatResult` has a basic variant (`Wait()` only) and a streaming variant (`StreamChatResult` with `Recv()`, `Wait()`, `Cancel()`). Streaming providers emit `StreamDelta` values (text, tool-call-start) through a channel; the Runner pulls deltas via `Recv()` loop and independently decides whether to forward to sink or fire events. This decouples provider streaming from sink delivery — e.g. Gemini streams at the provider level but content is filtered before user delivery (thinking leak protection). Events (emoji reactions) work for all providers regardless of streaming mode.

The `ProviderFactory` creates providers on demand, re-reading config each call. Providers enforce model whitelists. `SanitizeMessages()` removes orphaned tool messages before API calls.

### Tools (`tools/`)

Tools implement `Def() ToolDef` + `Run(ctx, args) string`. Registered in a `Registry`, cloned per-thread. Search and fetch tools use `SearchProvider`/`FetchProvider` interfaces with runtime `Available()` checks.

`dispatch` is the unified routing tool (6 targets: caller:user / caller:session / user / subagent / fork / session). The caller:* forms assert the actual caller kind — mismatches fail validation so the LLM can't silently misroute. `dispatch({})` with empty sends ends a turn silently. For delayed self-wakes (replacing the old `sleep_thread(duration=...)`), use the `manage-cron` skill to create a one-time `set-at --direct-wake` job into the current session.

### Audio Support

Audio recognition follows the same pattern as vision: `AudioModels` registered per provider, `SupportsAudio()` capability check, `<<media:audio/ogg:path>>` markers, and `audioreader` agent delegation for non-audio models.

- **Channel layer**: Telegram Voice/Audio and Discord audio attachments are downloaded to `{workspace}/media/` (same `downloadMedia()` as images).
- **Tool layer**: `DetectFileType` recognizes `FileTypeAudio` via extension + magic bytes. `handleAudio()` returns media marker if `SupportsAudio`, otherwise guides LLM to delegate to `audioreader`.
- **Provider layer**: OpenRouter sends audio markers as `input_audio` content parts. Gemini uses generic `inlineData`. Non-audio providers skip audio markers.
- **Token estimation**: `EstimateAudioTokens()` uses file size + bitrate heuristic, ~32 tokens/sec.
- **audioreader agent**: `specialty: audio`, configured during `onboard` (same flow as imagereader specialty routing).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [linanwx/nagobot](https://github.com/linanwx/nagobot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
