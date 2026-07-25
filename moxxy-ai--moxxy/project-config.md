---
trigger: always_on
description: You are working in the **moxxy** monorepo: a TypeScript framework for block-based, modular agentic loops. Every block — provider, loop strategy, tool, compactor, channel, skill, even the CLI — is swappable. Skills can synthesize new skills mid-session.
---

# AGENTS.md — Guide for AI agents working in this repo

You are working in the **moxxy** monorepo: a TypeScript framework for block-based, modular agentic loops. Every block — provider, loop strategy, tool, compactor, channel, skill, even the CLI — is swappable. Skills can synthesize new skills mid-session.

If you're a Claude Code agent or any other autonomous agent: read this file first, then jump to the workflow under `.ai/agents/` that matches your task.

> **Canonical AI catalog: `.ai/`.** Skills and agent author-guides live once under **`.ai/skills/`** and **`.ai/agents/`** — the single source of truth shared across assistants. `.claude/skills`, `.claude/agents`, `.codex/skills`, `.codex/agents` are **symlinks** into `.ai/`. Always edit the files under `.ai/`; never duplicate them under `.claude/` or `.codex/`, and don't chase the symlinked copies in circles. (`.claude/settings.json` and `.claude/hooks/` stay Claude-Code-specific and are NOT mirrored into `.ai/`.)

---

## Architecture, in 90 seconds

```
@moxxy/sdk     <— typed public surface (event types, define* helpers, hook signatures, provider/loop utils)
@moxxy/core    <— runtime (event log, plugin host, registries, permissions, session, skill loader)

@moxxy/tools-builtin              Read/Edit/Write/Bash/Grep/Glob + recall + Sleep
@moxxy/mode-default               "default" mode — Claude Code-style ReAct loop (registered by the CLI; first registered mode auto-activates)
@moxxy/mode-goal                  "goal" mode — autonomous auto-approve loop; works across turns until goal_complete
@moxxy/mode-deep-research         "research" mode — multi-query fan-out + synthesis
@moxxy/compactor-summarize        default summarize-old-turns compactor
@moxxy/cache-strategy-stable-prefix  default prompt-cache strategy (stable-prefix breakpoints; `none` = opt-out)
@moxxy/skills-builtin             MD skills shipped with the framework

@moxxy/plugin-provider-anthropic  first-party provider
@moxxy/plugin-provider-openai     OpenAI provider
@moxxy/plugin-vault               encrypted secret store (AES-256-GCM + keytar fallback)
@moxxy/plugin-memory              long-term memory journal + TF-IDF / vector recall
@moxxy/plugin-cli                 Ink TUI components + interactive PermissionResolver
@moxxy/plugin-telegram            Telegram channel (TOFU pairing)
@moxxy/plugin-channel-http        HTTP channel (auth + allow-list resolver)
@moxxy/plugin-channel-web         web surface channel — serves a browser app rendering agent-authored view-spec UIs over a WebSocket
@moxxy/plugin-channel-mobile      mobile channel — serves the desktop IPC contract over an authenticated WebSocket (`moxxy mobile`)
@moxxy/plugin-view                present_view tool — agent-authored JSX-like view-spec parsed into a validated AST channels can render
@moxxy/plugin-mcp                 MCP server bridge
@moxxy/plugin-embeddings-openai   OpenAI embeddings
@moxxy/plugin-embeddings-transformers   on-device embeddings via xenova
@moxxy/plugin-browser             web_fetch tool + Playwright sidecar (heavy)
@moxxy/plugin-scheduler           cron/heartbeat: time-driven prompts + auto-scheduled skills
@moxxy/plugin-workflows           swappable DAG engine: chain skills/prompts/tools into saved, schedulable pipelines
@moxxy/plugin-security            opt-in capability isolation: Isolator interface + `none` / `inproc` impls
@moxxy/isolator-worker            worker_threads-based Isolator (memory + time + JS-state isolation)
@moxxy/isolator-subprocess        subprocess Isolator (kernel-enforced process boundary)
@moxxy/isolator-wasm              WebAssembly Isolator (zero ambient authority; experimental)
@moxxy/plugin-provider-openai-codex  ChatGPT OAuth provider (Responses API)
@moxxy/plugin-provider-claude-code  Claude Pro/Max OAuth provider (Messages API, bearer token)
@moxxy/plugin-provider-admin      register OpenAI-compatible providers at runtime
@moxxy/plugin-oauth               generic OAuth 2.0 + PKCE / device-code
@moxxy/plugin-stt-whisper         Whisper transcriber (voice in); `-codex` variant reuses ChatGPT creds
@moxxy/plugin-computer-control    macOS native input (screenshot/click/type)
@moxxy/plugin-subagents           dispatch typed sub-agents from a turn
@moxxy/plugin-commands            built-in slash commands (/info, /clear, /compact, …)
@moxxy/plugin-self-update         agent edits its own plugins/skills (Tier 1) + core (Tier 2)
@moxxy/plugin-plugins-admin       install / remove / enable / disable plugins at runtime (model tools + `moxxy plugins` CLI + `/plugins` picker)
@moxxy/plugin-usage-stats         per-session token + cost accounting
@moxxy/plugin-webhooks            external-event triggers (verified HTTP listener + tunnels)
@moxxy/runner                     bare session runner; channels attach over a unix socket (JSON-RPC)

@moxxy/config      defineConfig + loader (cosmiconfig-style discovery + zod validation)
@moxxy/testing     FakeProvider + record/replay harness

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moxxy-ai/moxxy](https://github.com/moxxy-ai/moxxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
