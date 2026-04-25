---
trigger: always_on
description: Term Challenge is a WASM evaluation module for AI agents on the Bittensor network via platform-v2. Miners submit Python agent packages (as zip files) that solve SWE-bench tasks. The WASM module runs inside platform-v2 validators to validate submissions, evaluate task results, and compute scores. A companion native CLI (`term-cli`) provides a TUI for monitoring leaderboards, evaluation progress, and network health.
---

# AGENTS.md — Term Challenge

## Project Purpose

Term Challenge is a WASM evaluation module for AI agents on the Bittensor network via platform-v2. Miners submit Python agent packages (as zip files) that solve SWE-bench tasks. The WASM module runs inside platform-v2 validators to validate submissions, evaluate task results, and compute scores. A companion native CLI (`term-cli`) provides a TUI for monitoring leaderboards, evaluation progress, and network health.

## Architecture Overview

```
term-challenge/
├── Cargo.toml          # workspace with members = [".", "core", "wasm", "cli", "executor"]
├── src/
│   ├── lib.rs                  # Root library crate entry point
│   └── dataset/
│       ├── mod.rs              # Dataset module re-exports
│       ├── types.rs            # DatasetEntry struct (SWE-forge schema)
│       └── huggingface.rs      # HuggingFaceDataset: download, list, cache
├── wasm/
│   ├── Cargo.toml      # cdylib, depends on platform-challenge-sdk-wasm
│   └── src/
│       ├── lib.rs              # Challenge impl + register_challenge!
│       ├── types.rs            # Submission, TaskDefinition, AgentLogs, etc.
│       ├── scoring.rs          # Aggregate scoring, decay, weight calculation
│       ├── tasks.rs            # Active dataset storage (SWE-bench tasks)
│       ├── dataset.rs          # Dataset selection, consensus, and random index generation
│       ├── routes.rs           # Challenge route definitions and handlers for RPC
│       ├── agent_storage.rs    # Agent code, log, and evaluation status storage
│       ├── ast_validation.rs   # Python AST whitelist validation (imports, builtins, patterns)
│       ├── llm_review.rs       # LLM-based code review, reviewer selection, aggregation
│       ├── submission.rs       # Named submission registry and version tracking
│       └── timeout_handler.rs  # Review assignment timeout tracking and replacement
├── cli/
│   ├── Cargo.toml      # native binary, ratatui TUI
│   └── src/
│       ├── main.rs     # Entry point, event loop
│       ├── app.rs      # Application state
│       ├── ui.rs       # Ratatui UI rendering
│       └── rpc.rs      # JSON-RPC 2.0 client
├── docs/
│   ├── architecture.md
│   ├── miner/
│   │   ├── how-to-mine.md
│   │   └── submission.md
│   └── validator/
│       └── setup.md
├── .github/
│   └── workflows/
│       ├── ci.yml          # Build, clippy, test, WASM build, release on tags
│       └── release.yml     # release-please + artifact publishing
├── AGENTS.md
├── README.md
├── LICENSE
├── CHANGELOG.md
└── .githooks/
```

### Data Flow

1. **Miner** submits a zip package with agent code and task results
2. **RPC** receives submission, verifies signature, relays to validators
3. **Validators** run WASM `validate()` — checks signature, epoch rate limit, Basilica metadata, package size
4. **50% validator approval** → submission stored in blockchain
5. **Validators** run WASM `evaluate()`:
   a. **AST validation** — checks Python code against import whitelist, forbidden builtins, and dangerous patterns
   b. **LLM review** — optional LLM-based security review via executor proxy (`POST /llm/chat`) or host function (`host_llm_chat_completion`) if proxy URL not configured
   c. **Task scoring** — scores task results, optionally applies LLM judge per task
   d. **Aggregate & decay** — computes pass rate, applies epoch-based decay
6. **Agent code & logs** stored on-chain for auditability (code ≤ 1MB, logs ≤ 256KB)
7. **Log consensus** — validators propose logs, >50% hash agreement required
8. **Consensus** aggregates scores, applies decay, submits weights to Bittensor

### Key Concepts

- **WASM mode**: The `wasm32-unknown-unknown` module is loaded by platform-v2 validators
- **Host functions (WASM)**: WASM interacts with the outside world via `host_http_post()`, `host_storage_get()`, `host_storage_set()`, `host_consensus_get_epoch()`, `host_consensus_get_submission_count()`, `host_random_seed()`, `host_get_timestamp()`
- **SWE-bench datasets**: Tasks are selected from HuggingFace CortexLM/swe-bench via P2P consensus
- **Epoch rate limiting**: 1 submission per 3 epochs per miner
- **Top agent decay**: 60-epoch grace period, then exponential decay with 20-epoch half-life

## Agent Code Storage

Agent submissions are stored on-chain for auditability and retrieval. The `agent_storage` module manages three storage categories:

| Storage Key Format | Content | Max Size |
|---|---|---|
| `agent_code:<hotkey>:<epoch>` | Raw zip package bytes | 1 MB (1,048,576 bytes) |
| `agent_hash:<hotkey>:<epoch>` | Hash of the agent package | — |
| `agent_logs:<hotkey>:<epoch>` | Serialized `AgentLogs` struct | 256 KB (262,144 bytes) |

- **Package size limit**: Submissions with `package_zip` exceeding 1 MB are rejected at the storage layer.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PlatformNetwork/harness-challenge](https://github.com/PlatformNetwork/harness-challenge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
