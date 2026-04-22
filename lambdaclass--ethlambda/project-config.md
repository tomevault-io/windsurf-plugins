---
trigger: always_on
description: Development reference for ethlambda - minimalist Lean Ethereum consensus client.
---

# ethlambda Development Guide

Development reference for ethlambda - minimalist Lean Ethereum consensus client.
Not to be confused with Ethereum consensus clients AKA Beacon Chain clients AKA Eth2 clients.

## Quick Reference

**Main branch:** `main`
**Rust version:** 1.92.0 (edition 2024)
**Test fixtures commit:** Check `LEAN_SPEC_COMMIT_HASH` in Makefile

## Codebase Structure (10 crates)

```
bin/ethlambda/              # Entry point, CLI, orchestration
  └─ src/version.rs         # Build-time version info (vergen-git2)
crates/
  blockchain/               # State machine actor (GenServer pattern)
    ├─ src/lib.rs           # BlockChain actor, tick events, validator duties
    ├─ src/store.rs         # Fork choice store, block/attestation processing
    ├─ src/key_manager.rs   # Validator key management and signing
    ├─ src/metrics.rs       # Blockchain-level Prometheus metrics
    ├─ fork_choice/         # LMD GHOST implementation (3SF-mini)
    └─ state_transition/    # STF: process_slots, process_block, attestations
        └─ src/metrics.rs   # State transition timing + counters
  common/
    ├─ types/               # Core types (State, Block, Attestation, Checkpoint)
    ├─ crypto/              # XMSS aggregation (leansig wrapper)
    └─ metrics/             # Prometheus re-exports, TimingGuard, gather utilities
  net/
    ├─ p2p/                 # libp2p: gossipsub + req-resp (Status, BlocksByRoot)
    │   ├─ src/gossipsub/   # Topic encoding, message handling
    │   ├─ src/req_resp/    # Request/response codec and handlers
    │   └─ src/metrics.rs   # Peer connection/disconnection tracking
    └─ rpc/                 # Axum HTTP: API server + metrics server (independent ports)
  storage/                  # RocksDB backend, in-memory for tests
    └─ src/api/             # StorageBackend trait + Table enum
```

## Key Architecture Patterns

### Actor Concurrency (spawned-concurrency)
- **BlockChain**: Main state machine (GenServer pattern)
- **P2P**: Network event loop with libp2p swarm
- Communication via `mpsc::unbounded_channel`
- Shared storage via `Arc<dyn StorageBackend>` (clone Store, share backend)

### Tick-Based Validator Duties (4-second slots, 5 intervals per slot)
```
Interval 0: Block proposal → accept attestations if proposal exists
Interval 1: Attestation production (all validators, including proposer)
Interval 2: Aggregation (aggregators create proofs from gossip signatures)
Interval 3: Safe target update (fork choice)
Interval 4: Accept accumulated attestations
```

### Attestation Pipeline
```
Gossip → Signature verification → new_attestations (pending)
  ↓ (intervals 0/4)
promote → known_attestations (fork choice active)
  ↓
Fork choice head update
```

### State Transition Phases
1. **process_slots()**: Advance through empty slots, update historical roots
2. **process_block()**: Validate header → process attestations → update justifications/finality
3. **Justification**: 3SF-mini rules (delta ≤ 5 OR n² OR n(n+1))
4. **Finalization**: Source with no unjustifiable gaps to target

## Development Workflow

### Before Committing
```bash
make fmt                                     # Format code (cargo fmt --all)
make lint                                    # Clippy with -D warnings
make test                                    # All tests + forkchoice spec tests
```

### Common Operations
```bash
.claude/skills/test-pr-devnet/scripts/test-branch.sh    # Test branch in multi-client devnet
rm -rf leanSpec && make leanSpec/fixtures                # Regenerate test fixtures (requires uv)
make docker-build                                        # Build Docker image (DOCKER_TAG=local)
make run-devnet                                          # Run local devnet with lean-quickstart
```

### Testing with Local Devnet

See `.claude/skills/test-pr-devnet/SKILL.md` for multi-client devnet testing workflows.

## Important Patterns & Idioms

### Trait Implementations
```rust
// Prefer From/Into traits over custom from_x/to_x methods
impl From<u8> for ResponseCode { fn from(code: u8) -> Self { Self(code) } }
impl From<ResponseCode> for u8 { fn from(code: ResponseCode) -> Self { code.0 } }

// Enables idiomatic .into() usage
let code: ResponseCode = byte.into();
let byte: u8 = code.into();
```

### Ownership for Large Structures
```rust
// Prefer taking ownership to avoid cloning large data (signatures ~3KB)
pub fn insert_signed_block(&mut self, root: H256, signed_block: SignedBlock) { ... }

// Add .clone() at call site if needed - makes cost explicit
store.insert_signed_block(block_root, signed_block.clone());
```

### Formatting Patterns
```rust
// Extract long arguments into variables so formatter can join lines
// Instead of:
batch.put_batch(Table::X, vec![(key, value)]).expect("msg");

// Prefer:
let entries = vec![(key, value)];
batch.put_batch(Table::X, entries).expect("msg");
```

### Error Handling Patterns

**Use `inspect` and `inspect_err` for side-effect-only error handling:**
```rust
// ✅ GOOD: Use inspect_err when only logging or performing side effects on error
result
    .inspect_err(|err| warn!(%err, "Operation failed"));

// Extract complex expressions to variables for cleaner formatting

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lambdaclass/ethlambda](https://github.com/lambdaclass/ethlambda) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
