---
trigger: always_on
description: **term-executor** is a remote evaluation executor for the [term-challenge](https://github.com/PlatformNetwork/term-challenge) platform. It runs as a containerized Rust service on [Basilica](https://basilica.ai) that receives batch task archives via multipart upload, executes agent code against cloned task repositories, runs validation test scripts, and reports pass/fail results with aggregate rewards. It is the core compute backend that evaluates AI agent coding challenges.
---

# AGENTS.md — term-executor

## Project Purpose

**term-executor** is a remote evaluation executor for the [term-challenge](https://github.com/PlatformNetwork/term-challenge) platform. It runs as a containerized Rust service on [Basilica](https://basilica.ai) that receives batch task archives via multipart upload, executes agent code against cloned task repositories, runs validation test scripts, and reports pass/fail results with aggregate rewards. It is the core compute backend that evaluates AI agent coding challenges.

## Architecture Overview

This is a **single-crate Rust binary** (`term-executor`) built with Axum. There are no sub-crates or workspaces.

### Data Flow

```
Validator → POST /submit (multipart archive) → term-executor
  1. Authenticate via X-Hotkey, X-Nonce, X-Signature headers
  2. Verify hotkey is in the dynamic validator whitelist (Bittensor netuid 100, >10k TAO stake)
  3. Compute SHA-256 hash of archive bytes
  4. Record vote in ConsensusManager
  5. If <50% of whitelisted validators have voted for this hash:
     → Return 202 Accepted with pending_consensus status
  6. If ≥50% consensus reached:
     a. Extract uploaded archive (zip/tar.gz) containing tasks/ and agent_code/
     b. Parse each task: workspace.yaml, prompt.md, tests/
     c. For each task (concurrently, up to limit):
        i.   git clone the target repository at base_commit
        ii.  Run install commands (pip install, etc.)
        iii. Write & execute agent code in the repo
        iv.  Write test source files into the repo
        v.   Run test scripts (bash), collect exit codes
     d. Aggregate results (reward per task, aggregate reward)
     e. Stream progress via WebSocket (GET /ws?batch_id=...)
     f. Return results via GET /batch/{id}
```

### Background Tasks

```
ValidatorWhitelist refresh loop (every 5 minutes):
  1. Connect to Bittensor subtensor via BittensorClient::with_failover()
  2. Sync metagraph for netuid 100
  3. Filter validators: validator_permit && active && stake >= 10,000 TAO
  4. Atomically replace whitelist with new set of SS58 hotkeys
  5. On failure: retry up to 3 times with exponential backoff, keep cached whitelist

ConsensusManager reaper loop (every 30 seconds):
  1. Remove pending consensus entries older than TTL (default 60s)
```

### Module Map

| File | Responsibility |
|---|---|
| `src/main.rs` | Entry point — bootstraps config, session manager, executor, validator whitelist, consensus manager, Axum server, background tasks |
| `src/config.rs` | `Config` struct loaded from environment variables with defaults; Bittensor and consensus configuration |
| `src/handlers.rs` | Axum route handlers: `/health`, `/status`, `/metrics`, `/submit`, `/batch/{id}`, `/batch/{id}/tasks`, `/batch/{id}/task/{task_id}`, `/batches` |
| `src/auth.rs` | Authentication: `extract_auth_headers()`, `verify_request()` (whitelist-based), `validate_ss58()`, sr25519 signature verification via `verify_sr25519_signature()`, SS58 checksum via `blake2`, `NonceStore` for replay protection, `AuthHeaders`/`AuthError` types |
| `src/validator_whitelist.rs` | Dynamic validator whitelist — fetches validators from Bittensor netuid 100 every 5 minutes, filters by stake ≥10k TAO, stores SS58 hotkeys in `parking_lot::RwLock<HashSet>` |
| `src/consensus.rs` | 50% consensus manager — tracks pending votes per archive hash in `DashMap`, triggers evaluation when ≥50% of whitelisted validators submit same payload, TTL reaper for expired entries |
| `src/executor.rs` | Core evaluation engine — spawns batch tasks that clone repos, run agents, run tests concurrently |
| `src/session.rs` | `SessionManager` with `DashMap`, `Batch`, `BatchResult`, `TaskResult`, `BatchStatus`, `TaskStatus`, `WsEvent` types |
| `src/task.rs` | Archive extraction (zip/tar.gz), task directory parsing, agent code loading, language detection |
| `src/metrics.rs` | Atomic counter-based Prometheus metrics (batches total/active/completed, tasks passed/failed, duration) |
| `src/cleanup.rs` | Work directory removal, stale session reaping, process group killing |
| `src/ws.rs` | WebSocket handler for real-time batch progress streaming |

### Key Shared State (via `Arc`)

- `AppState` (in `handlers.rs`) holds `Config`, `SessionManager`, `Metrics`, `Executor`, `NonceStore`, `started_at`, `ValidatorWhitelist`, `ConsensusManager`
- `SessionManager` uses `DashMap<String, Arc<Batch>>` for lock-free concurrent access
- `ValidatorWhitelist` uses `parking_lot::RwLock<HashSet<String>>` for concurrent read access with rare writes
- `ConsensusManager` uses `DashMap<String, PendingConsensus>` for lock-free concurrent vote tracking
- Per-batch `Semaphore` in `executor.rs` controls concurrent tasks within a batch (configurable, default: 8)
- `broadcast::Sender<WsEvent>` per batch for WebSocket event streaming

## Tech Stack


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BaseIntelligence/term-executor](https://github.com/BaseIntelligence/term-executor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
