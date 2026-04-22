---
trigger: always_on
description: Atomic is a personal knowledge base that turns freeform markdown notes ("atoms") into a semantically-connected, AI-augmented knowledge graph. It runs as a Tauri desktop app, a headless HTTP server, or both simultaneously.
---

# Atomic

Atomic is a personal knowledge base that turns freeform markdown notes ("atoms") into a semantically-connected, AI-augmented knowledge graph. It runs as a Tauri desktop app, a headless HTTP server, or both simultaneously.

# You

You are an expert software developer and architect. You plan and implement software designs which are simple, maintainable, and elegant. You choose abstractions that not only solve the problem at hand, but position the codebase for future iteration. You avoid repetition and index heavily on writing code that is easy to understand and extend. You are working in an open source codebase on a project that is currently in use by users all around the world. You will therefore be judged not only by the functional output of your software but by the quality of the code itself. Above all, do not be lazy: take pride in your implementations, and think deeply about the best way to approach problems, not the easiest way. Quality is everything.

## Core Concepts

**Atoms** are the fundamental unit — markdown notes with optional source URLs and hierarchical tags. When an atom is created or updated, an asynchronous pipeline automatically:
1. Chunks the content using markdown-aware boundaries (respecting code blocks, headers, paragraphs)
2. Generates vector embeddings via the configured AI provider
3. Extracts and assigns tags using LLM structured outputs (if auto-tagging is enabled)
4. Builds semantic edges to other atoms based on embedding similarity

This pipeline is fire-and-forget from the caller's perspective — the caller receives the saved atom immediately while embedding/tagging runs in the background, with progress reported via callbacks.

**Tags** form a hierarchical tree. Auto-extracted tags are organized under category parents (Topics, People, Locations, Organizations, Events). Tags serve as both organizational structure and scoping mechanism for wiki generation and chat conversations.

**Wiki articles** are LLM-synthesized summaries of all atoms under a given tag, with inline citations linking back to source atoms. They support incremental updates — when new atoms are tagged, only the new content is sent to the LLM to integrate into the existing article.

**Chat** is an agentic RAG system. Conversations can be scoped to specific tags, and the agent has tools to search the knowledge base semantically during conversation. Responses stream back through the same callback system used by embeddings.

**Canvas** is a spatial visualization where atoms are positioned using d3-force simulation. Atoms sharing tags are linked, and a custom similarity force pulls semantically-related atoms together. Positions are persisted so the layout is stable across sessions.

## Architecture: Core + Thin Wrappers

The central architectural principle is the separation of **business logic** from **transport**. All domain logic lives in `atomic-core`, a standalone Rust crate with no framework dependencies. Every client is a thin wrapper that adapts `atomic-core` to a specific transport mechanism.

```
                    ┌─────────────────┐
                    │   atomic-core   │
                    │  (all logic)    │
                    └────────┬────────┘
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
    ┌─────────────┐  ┌──────────────┐  ┌───────────┐
    │  src-tauri   │  │atomic-server │  │ mcp-bridge│
    │ (Tauri IPC)  │  │(REST+WS+MCP)│  │(stdio→HTTP)│
    └──────┬──────┘  └──────┬───────┘  └─────┬─────┘
           │                │                 │
    ┌──────▼──────┐  ┌──────▼───────┐        │
    │   React UI   │  │  HTTP clients│  ┌─────▼──────┐
    │(Tauri or HTTP)│  │ (iOS, etc.) │  │ MCP clients│
    └─────────────┘  └──────────────┘  │(Claude,etc)│
                                       └────────────┘
```

### `atomic-core` — The Facade

`AtomicCore` is a `Clone` wrapper around `Arc<Database>` that exposes every operation: CRUD, search, embedding, wiki generation, chat, clustering, tag compaction, and import. It is completely transport-agnostic.

The key design decision is **callback-based eventing**: operations that produce async events (embedding, chat) accept `Fn(EmbeddingEvent)` or `Fn(ChatEvent)` closures. The core doesn't know or care how events are delivered — it just calls the closure. This makes it usable from any Rust context without pulling in Tauri, actix, or any framework.

### `src-tauri` — Desktop Wrapper

The Tauri app spawns `atomic-server` as a **sidecar process** and exposes a single IPC command (`get_local_server_config`) that returns the server's base URL and auth token. The frontend then connects to the sidecar over HTTP/WebSocket, exactly as it would to a standalone `atomic-server`. On exit, Tauri kills the sidecar.

### `atomic-server` — Headless HTTP Wrapper

The standalone server wraps `atomic-core` with a full REST API (~78 routes) plus a WebSocket endpoint and a Streamable HTTP MCP endpoint. The same thin-wrapper pattern applies: each route handler unpacks HTTP request params, calls `core.method()`, returns JSON.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kenforthewin/atomic](https://github.com/kenforthewin/atomic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
