---
trigger: always_on
description: OpenClaudia is an open-source universal agent harness that provides Claude Code-like capabilities for any AI. It acts as a proxy server translating between OpenAI-compatible formats and multiple provider APIs (Anthropic, OpenAI, Google Gemini, DeepSeek, Qwen, Z.AI/GLM).
---

# CLAUDE.md - OpenClaudia Development Guide

## Project Overview

OpenClaudia is an open-source universal agent harness that provides Claude Code-like capabilities for any AI. It acts as a proxy server translating between OpenAI-compatible formats and multiple provider APIs (Anthropic, OpenAI, Google Gemini, DeepSeek, Qwen, Z.AI/GLM).

## Architecture Map

```
                              ┌─────────────────────────────────────────────────────────────┐
                              │                      OpenClaudia                            │
                              └─────────────────────────────────────────────────────────────┘
                                                          │
                    ┌─────────────────────────────────────┼─────────────────────────────────┐
                    │                                     │                                 │
                    ▼                                     ▼                                 ▼
            ┌───────────────┐                    ┌───────────────┐                 ┌───────────────┐
            │    main.rs    │                    │    tui.rs     │                 │    web.rs     │
            │  CLI Entry    │                    │  Terminal UI  │                 │  Web Scraping │
            │  (clap)       │                    │  (ratatui)    │                 │  (headless)   │
            └───────┬───────┘                    └───────────────┘                 └───────────────┘
                    │
        ┌───────────┼───────────┬───────────────────────┬───────────────────────┐
        │           │           │                       │                       │
        ▼           ▼           ▼                       ▼                       ▼
┌───────────┐ ┌───────────┐ ┌───────────┐       ┌───────────────┐       ┌───────────────┐
│ config.rs │ │ proxy.rs  │ │ session.rs│       │   hooks.rs    │       │   rules.rs    │
│ YAML +    │ │ HTTP Proxy│ │ State Mgmt│       │ Pre/Post Tool │       │ CLAUDE.md     │
│ Env Vars  │ │ (axum)    │ │ Turns     │       │ Lifecycle     │       │ .clauderules  │
└─────┬─────┘ └─────┬─────┘ └───────────┘       └───────────────┘       └───────────────┘
      │             │
      │             ▼
      │     ┌───────────────────────────────────────────────────────────────────────────┐
      │     │                           providers.rs                                    │
      │     │                    ProviderAdapter trait + Implementations                │
      │     └───────────────────────────────────────────────────────────────────────────┘
      │                                         │
      │         ┌─────────────┬─────────────┬───┴───────┬─────────────┬─────────────┐
      │         ▼             ▼             ▼           ▼             ▼             ▼
      │    ┌─────────┐   ┌─────────┐   ┌─────────┐ ┌─────────┐   ┌─────────┐   ┌─────────┐
      │    │Anthropic│   │ OpenAI  │   │ Google  │ │DeepSeek │   │  Qwen   │   │  Z.AI   │
      │    │ Adapter │   │ Adapter │   │ Gemini  │ │ Adapter │   │ Adapter │   │  GLM    │
      │    └─────────┘   └─────────┘   └─────────┘ └─────────┘   └─────────┘   └─────────┘
      │
      ▼
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                                     tools.rs                                            │
│        bash | read | write | edit | glob | grep | web_fetch | memory_* | chainlink     │
└─────────────────────────────────────────────────────────────────────────────────────────┘
              │                                           │
              ▼                                           ▼
      ┌───────────────┐                           ┌───────────────┐
      │   memory.rs   │                           │ compaction.rs │
      │ SQLite Store  │                           │ Context Mgmt  │
      │ Core/Archival │                           │ Summarization │
      └───────────────┘                           └───────────────┘
              │
              ▼
      ┌───────────────┐       ┌───────────────┐
      │   mcp.rs      │       │  plugins.rs   │
      │ MCP Protocol  │       │ Future Ext    │
      │ (stdio/http)  │       │ Framework     │
      └───────────────┘       └───────────────┘
```

## Module Responsibilities

| Module          | Purpose                                                    |
|-----------------|-----------------------------------------------------------|
| `main.rs`       | CLI entry point, subcommands (init, start, chat, loop)    |
| `config.rs`     | YAML config + env var loading, provider/hook definitions  |
| `proxy.rs`      | HTTP server (axum), request/response translation          |
| `providers.rs`  | Provider adapters (Anthropic, OpenAI, Google, etc.)       |
| `tools.rs`      | Tool definitions and execution (bash, read, write, edit)  |
| `memory.rs`     | SQLite-based archival + core memory (MemGPT-style)        |
| `session.rs`    | Conversation state, turn management                        |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dollspace-gay/OpenClaudia](https://github.com/dollspace-gay/OpenClaudia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
