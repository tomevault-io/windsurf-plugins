---
trigger: always_on
description: Sage is a privacy-first personal AI agent with persistent memory, built in Rust. It communicates via Signal (E2E encrypted), runs LLM inference in TEEs (Trusted Execution Environments) via Maple, and stores all data locally in PostgreSQL with pgvector. The agent uses a 4-tier memory architecture inspired by Letta/MemGPT and typed DSRs signatures instead of native LLM tool calling.
---

# AGENTS.md - Sage

## Project Overview

Sage is a privacy-first personal AI agent with persistent memory, built in Rust. It communicates via Signal (E2E encrypted), runs LLM inference in TEEs (Trusted Execution Environments) via Maple, and stores all data locally in PostgreSQL with pgvector. The agent uses a 4-tier memory architecture inspired by Letta/MemGPT and typed DSRs signatures instead of native LLM tool calling.

## Repository Structure

```
sage/
├── Cargo.toml                  # Workspace root (resolver = "2")
├── Cargo.lock
├── rust-toolchain.toml         # Stable toolchain with rustfmt, clippy, rust-src
├── Dockerfile                  # Multi-stage build (cargo-chef + debian:bookworm-slim)
├── docker-compose.yml          # Full stack: postgres, signal-cli, sage
├── flake.nix                   # Nix dev shell (provides podman, diesel-cli, signal-cli, etc.)
├── flake.lock
├── justfile                    # Task runner (all build/deploy/dev commands)
├── .env.example                # Environment variable template
├── .github/
│   └── workflows/
│       ├── ci.yml              # Check, test, fmt, clippy (on push/PR to master)
│       ├── release.yml         # Build binaries on tag push (v*)
│       └── docker-publish.yml  # Build + push multi-arch images to ghcr.io
├── .githooks/
│   └── pre-commit              # Runs fmt, clippy, test before commit
├── examples/
│   └── gepa/
│       ├── trainset.json       # GEPA training examples
│       └── valset.json         # GEPA validation examples
├── optimized_instructions/     # GEPA-optimized agent instructions
├── docs/                       # Design docs, architecture notes, prompt examples
└── crates/
    ├── sage-core/              # Main application crate
    │   ├── Cargo.toml
    │   ├── migrations/         # Diesel PostgreSQL migrations (11 total)
    │   ├── src/
    │   │   ├── main.rs         # Entry point: tokio runtime, event loop, Signal listener
    │   │   ├── lib.rs          # Public API re-exports
    │   │   ├── config.rs       # Config struct from environment variables
    │   │   ├── sage_agent.rs   # Core agent: DSRs signatures, tool registry, step loop
    │   │   ├── agent_manager.rs# Multi-user agent management with isolated memory
    │   │   ├── signal.rs       # Signal JSON-RPC client (TCP + subprocess modes)
    │   │   ├── tools.rs        # DoneTool, WebSearchTool implementations
    │   │   ├── shell_tool.rs   # Shell command execution with safety checks
    │   │   ├── vision.rs       # Image description via vision LLM pre-processing
    │   │   ├── scheduler.rs    # Cron + one-off task scheduling (PostgreSQL-backed)
    │   │   ├── scheduler_tools.rs # schedule_task, list_schedules, cancel_schedule tools
    │   │   ├── storage.rs      # Basic Diesel message storage
    │   │   ├── schema.rs       # Diesel schema (agents, blocks, messages, passages, summaries, etc.)
    │   │   ├── memory/
    │   │   │   ├── mod.rs      # MemoryManager: coordinates all 4 memory tiers
    │   │   │   ├── block.rs    # Core memory blocks (persona, human) - always in context
    │   │   │   ├── recall_new.rs   # Recall memory: conversation history with embeddings
    │   │   │   ├── archival_new.rs # Archival memory: long-term semantic storage (pgvector)
    │   │   │   ├── compaction.rs   # Summary/compaction when context window fills
    │   │   │   ├── context.rs  # Context window management and token estimation
    │   │   │   ├── db.rs       # Database operations for all memory tiers
    │   │   │   ├── embedding.rs# Embedding service (Maple TEE nomic-embed-text)
    │   │   │   └── tools.rs    # Memory manipulation tools for the agent
    │   │   └── bin/
    │   │       └── gepa_optimize.rs # GEPA prompt optimization CLI (~700 lines)
    └── sage-tools/             # External tool integrations
        ├── Cargo.toml
        └── src/
            ├── lib.rs          # ToolResult type, re-exports
            ├── brave.rs        # Brave Search API client (Pro) (~740 lines)
            └── web_search.rs   # WebSearch tool wrapper
```

## Development Environment

### Prerequisites

- [Nix](https://nixos.org/download.html) with flakes enabled (provides the full dev environment)
- Alternatively: Rust stable toolchain, podman/docker, diesel-cli, PostgreSQL, signal-cli

### Nix Dev Shell

The project uses a Nix flake for reproducible development. The flake provides:
- Rust stable toolchain (rustc, cargo, clippy, rustfmt, rust-analyzer)
- Podman + container tools (Linux only: conmon, slirp4netns, fuse-overlayfs)
- PostgreSQL with pgvector extension
- diesel-cli for database migrations
- signal-cli (standalone ARM64 binary on aarch64-linux, nixpkgs elsewhere)
- jq, just, valkey

```bash
# Enter the dev shell
nix develop

# Or run a single command inside the shell
nix develop --command just build
```

On Linux, the shell hook aliases `docker` to `podman` and configures container policy/runtime automatically.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AnthonyRonning/sage](https://github.com/AnthonyRonning/sage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
