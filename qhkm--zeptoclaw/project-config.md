---
trigger: always_on
description: Ultra-lightweight personal AI assistant. Fresh configs default to `assistant` mode with dangerous tool approvals enabled.
---

# ZeptoClaw

Ultra-lightweight personal AI assistant. Fresh configs default to `assistant` mode with dangerous tool approvals enabled.

## Quick Reference

```bash
cargo build --release                      # Build
cargo nextest run --lib                    # Test (use nextest to avoid OOM)
cargo clippy -- -D warnings && cargo fmt   # Lint & format
./target/release/zeptoclaw agent -m "Hello"  # Run agent
./target/release/zeptoclaw config check      # Validate config
./target/release/zeptoclaw provider status   # Check providers
```

For full CLI reference, slash commands, and gateway commands see `docs/claude/commands.md`.

## Agent Workflow — Task Tracking Protocol

Every Claude Code session MUST follow these rules:

### 1. Session Start — Check open issues
```bash
gh issue list --state open --limit 20
```
Present issues, ask what to work on.

### 2. New Work — Create issue first
```bash
gh issue create \
  --title "feat: short description" \
  --label "feat,area:tools" \
  --body "Brief description of the work."
```
Labels: `bug`, `feat`, `rfc`, `chore`, `docs` + `area:tools`, `area:channels`, `area:providers`, `area:safety`, `area:config`, `area:cli`, `area:memory` + `P1-critical`, `P2-high`, `P3-normal`. Skip for trivial changes.

### 3. Session End — Link and close
- Follow the PR guidelines in `docs/claude/pr.md` and use the template at `.github/PULL_REQUEST_TEMPLATE.md`
- PR body: include `Closes #N`
- **NEVER merge PRs without explicit user approval.** Wait for CI, present URL, merge only after user says to
- Merge: `gh pr merge <number> --squash --delete-branch --admin`
- Direct commit: `gh issue close N --comment "Done in <commit-sha>"`
- Update `CLAUDE.md` and `AGENTS.md` per the post-implementation checklist

## Pre-Push Checklist (MANDATORY)

```bash
cargo fmt && cargo clippy -- -D warnings && cargo nextest run --lib && cargo test --doc && cargo fmt -- --check
```

**After subagent work:** ALWAYS run `cargo fmt` before committing.

## Architecture

```
src/
├── agent/       # Agent loop, context builder, token budget, compaction
├── api/         # Panel API server + OpenAI-compatible serve routes (axum)
├── auth/        # OAuth (PKCE), token refresh, Claude CLI import
├── bus/         # Async message bus
├── channels/    # Telegram, Slack, Discord, Webhook, WhatsApp, Lark, Email, MQTT, Serial
├── cli/         # Clap commands + handlers
├── config/      # Config types/loading + hot-reload
├── cron/        # Persistent cron scheduler
├── deps/        # Dependency manager
├── gateway/     # Containerized agent proxy
├── health.rs    # Health server + metrics
├── memory/      # Workspace + long-term memory (pluggable search)
├── peripherals/ # Hardware: GPIO, I2C, NVS (ESP32, RPi, Arduino)
├── providers/   # Claude, OpenAI, Retry, Fallback, Quota
├── runtime/     # Native, Docker, Apple, Landlock, Firejail, Bubblewrap
├── routines/    # Event/webhook/cron automations
├── r8r_bridge/  # WebSocket bridge for r8r workflow approvals
├── safety/      # Injection detection, leak scanning, policy engine
├── security/    # Shell blocklist, path validation, secret encryption
├── session/     # Session persistence, history, auto-repair
├── tools/       # 33 built-in + MCP + plugins + android
├── utils/       # sanitize, metrics, telemetry, cost
└── main.rs      # Entry point → cli::run()

panel/           # React + Vite dashboard
landing/         # Static landing page
```

For detailed module docs see `docs/claude/architecture.md`.

## Coding Core Notes

- Embedded `ZeptoAgent` tool calls use the same `kernel::execute_tool()` path as the main agent loop and MCP server, so safety scanning, taint checks, and tool metrics stay aligned across entry points.
- Embedded `ZeptoAgent` also supports per-tool timeout, panic capture, and configurable approval gating via the builder for safer embedded coding-agent execution.
- The `panel` CLI namespace is always parsed, but panel-backed behavior still requires the optional Cargo `panel` feature; feature-disabled builds now fail with explicit build/install guidance instead of a Clap unknown-subcommand error.
- Model-driven provider inference treats vendor-prefixed gateway IDs like `anthropic/...` as OpenRouter models only when OpenRouter is actually available, and live provider model discovery now carries `api-version` while normalizing Azure deployment bases to `/openai/models`.
- The OpenAI-compatible `/v1/chat/completions` serve path forwards request tools, returns OpenAI-style tool-call payloads for assistant/tool messages, and the default provider streaming adapter now emits a text delta plus tool-call events before `Done` so non-native streaming providers are not silently flattened.
- The serve API only accepts omitted, `null`, or `"auto"` for `tool_choice`; unsupported values are rejected with `400` instead of being ignored.
- `shell` tool output is truncated at 2,000 lines / 50KB before it reaches the model context.
- `grep` reports subprocess failures instead of collapsing them into "No matches found".
- `edit_file` rejects empty `old_text` and accepts optional `expected_replacements` to guard exact-match edits.

## Common Tasks

### Add a new provider/tool/channel

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qhkm/zeptoclaw](https://github.com/qhkm/zeptoclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
