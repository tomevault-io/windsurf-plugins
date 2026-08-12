---
trigger: always_on
description: > This file defines conventions, architecture rules, and workflows for AI-assisted development of Ghost.
---

# Ghost — Agent Development Instructions

> This file defines conventions, architecture rules, and workflows for AI-assisted development of Ghost.
> All agents (Claude, Copilot, or any AI assistant) MUST follow these instructions.

---

## Project Overview

**Ghost** is a private, local-first **Agent OS** for desktop (Windows → macOS → Linux) and mobile (Android → iOS). It indexes local files, provides hybrid semantic + keyword search, connects to thousands of tools via open protocols (MCP, A2A, AG-UI, A2UI, WebMCP), and evolves into a full agent that takes actions on your behalf — all without sending data to the cloud.

- **Current phase**: Phase 1.7 — Multiplatform (backend ✅, frontend ✅, Android APK ✅, iOS needs macOS)
- **Stack**: Tauri v2 (Rust backend) + React/TypeScript (frontend) + SQLite/sqlite-vec + Candle (native AI) + rmcp (MCP SDK)
- **Repo**: `ghostapp-ai/ghost` (public, MIT)
- **Priority**: MCP Apps, Skills.md, then A2A/WebMCP protocols.
- **Protocol stack**: MCP (tools) → AG-UI (agent↔user streaming) → A2UI (generative UI) → A2A (multi-agent) → WebMCP (web agents)
- **Platforms**: Desktop (Windows, macOS, Linux) + Mobile (Android, iOS) via Tauri v2 conditional compilation

---

## Critical Rules

### 1. Never Break Privacy
- NEVER add telemetry, analytics, or any external network calls (except to localhost Ollama)
- NEVER include tracking pixels, error reporting services, or crash analytics
- All data processing MUST happen locally
- If a feature requires cloud access, it MUST be opt-in and clearly documented

### 2. Performance is Non-Negotiable
- App cold start: <500ms
- FTS5 keyword search: <5ms
- Semantic vector search: <500ms
- Idle RAM: <40MB
- Background indexing CPU: <10%
- Always benchmark before and after changes that touch search or indexing

### 3. Always Update the 3 Core Documents
After every significant change, update these files to reflect current state:
- **README.md** — Project description, features, architecture, getting started
- **ROADMAP.md** — Check off completed items, add new tasks discovered during implementation
- **CLAUDE.md** — Update conventions, add new patterns, document decisions

### 4. Research Before Implementing
- Before using a new crate or npm package, verify it exists and check its latest version
- Validate compatibility with our stack (Tauri v2, Rust 2021 edition, React 18)
- Check for security advisories
- Prefer well-maintained crates with >100 GitHub stars

### 5. Commits Must Be Professional
- Use conventional commits: `feat:`, `fix:`, `refactor:`, `docs:`, `test:`, `chore:`
- Each commit should be atomic — one logical change per commit
- Always include what changed and why in the commit message
- Never commit secrets, API keys, or sensitive data

---

## Architecture Rules

### Rust Backend (`src-tauri/`)

```
src-tauri/src/
├── lib.rs              # Tauri app builder, plugin registration, command handlers
├── main.rs             # Entry point (DO NOT modify beyond run())
├── indexer/
│   ├── mod.rs          # Public API for indexing module
│   ├── watcher.rs      # File system watcher (notify crate)
│   ├── extractor.rs    # Text extraction (PDF, DOCX, XLSX, TXT)
│   └── chunker.rs      # Text chunking strategy (512 tokens, 64 overlap)
├── db/
│   ├── mod.rs          # Database initialization, migrations, and CRUD operations
│   └── schema.rs       # Table definitions and migrations
├── embeddings/
│   ├── mod.rs          # EmbeddingEngine (fallback chain: Native → Ollama → None)
│   ├── native.rs       # Candle-based in-process BERT inference (all-MiniLM-L6-v2)
│   ├── ollama.rs       # OllamaEngine HTTP client (fallback engine)
│   └── hardware.rs     # Hardware detection (CPU cores, SIMD, GPU backend, RAM)
├── chat/
│   ├── mod.rs          # ChatEngine orchestration, model lifecycle, Ollama fallback
│   ├── native.rs       # llama-cpp-2 GGUF inference (Qwen2.5/3-Instruct, desktop-only)
│   ├── models.rs       # Model registry, auto-selection, HF Hub cache detection
│   └── inference.rs    # Hardware-adaptive inference profiles (GPU layers, threads, batch size)
├── search/
│   ├── mod.rs          # Search engine combining FTS5 + vector
│   └── ranking.rs      # RRF (Reciprocal Rank Fusion) implementation
├── protocols/          # (Phase 1.5+) Protocol Hub — all agent protocols
│   ├── mod.rs          # Protocol registry, initialization
│   ├── mcp_server.rs   # Ghost as MCP server (rmcp ServerHandler)
│   ├── mcp_client.rs   # Ghost connects to external MCP servers (rmcp ClientHandler)
│   ├── mcp_catalog.rs  # Curated MCP catalog (30+ servers) + one-click install + runtime detection
│   ├── runtime_bootstrap.rs # Zero-config runtime installer (Node.js, uv/Python, Docker detection)
│   ├── agui.rs         # AG-UI event system (30+ event types, broadcast bus, SSE endpoint)
│   ├── a2ui.rs         # A2UI v0.9 generative UI types + component builders (Google spec)
│   └── a2a.rs          # A2A v0.3.0 types (AgentCard, Task, JSON-RPC) + stub dispatcher
├── agent/              # (Phase 1.5+) Agent Engine — ReAct loop + tool calling
│   ├── mod.rs          # Shared types (ToolCall, OllamaTool, AgentRunResult, ExecutedToolCall)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elias489/ghost](https://github.com/elias489/ghost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
