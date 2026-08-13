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
            │    main.rs    │                    │    tui/       │                 │    web.rs     │
            │  CLI Entry    │                    │  Terminal UI  │                 │  Web Scraping │
            │  (clap)       │                    │  (ratatui)    │                 │  (headless)   │
            └───────┬───────┘                    └───────────────┘                 └───────────────┘
                    │
        ┌───────────┼───────────┬───────────────────────┬───────────────────────┐
        │           │           │                       │                       │
        ▼           ▼           ▼                       ▼                       ▼
┌───────────┐ ┌───────────┐ ┌───────────┐       ┌───────────────┐       ┌───────────────┐
│ config/   │ │ proxy.rs  │ │ session/  │       │   hooks/      │       │   rules.rs    │
│ YAML +    │ │ HTTP Proxy│ │ State Mgmt│       │ Pre/Post Tool │       │ CLAUDE.md     │
│ Env Vars  │ │ (axum)    │ │ Turns     │       │ Lifecycle     │       │ .clauderules  │
└─────┬─────┘ └─────┬─────┘ └───────────┘       └───────────────┘       └───────────────┘
      │             │
      │             ▼
      │     ┌───────────────────────────────────────────────────────────────────────────┐
      │     │                           providers/                                     │
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
│                                     tools/                                              │
│        bash | read | write | edit | glob | grep | web_fetch | memory_* | crosslink     │
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
      │   mcp.rs      │       │  plugins/     │
      │ MCP Protocol  │       │ Extension     │
      │ (stdio/http)  │       │ Framework     │
      └───────────────┘       └───────────────┘
```

## Module Responsibilities

Most subsystems are directory modules (e.g. `config/mod.rs`), not single
files. Single-file modules keep the `.rs` suffix below.

| Module          | Purpose                                                    |
|-----------------|-----------------------------------------------------------|
| `main.rs`       | CLI entry point, subcommands (init, start, chat, loop)    |
| `cli/`          | Subcommand implementations (auth, init, start, etc.)      |
| `config/`       | YAML config + env var loading, provider/hook definitions  |
| `proxy.rs`      | HTTP server (axum), request/response translation          |
| `providers/`    | Provider adapters (Anthropic, OpenAI, Google, etc.)       |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Corvidae-Coding-Projects/OpenClaudia](https://github.com/Corvidae-Coding-Projects/OpenClaudia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
