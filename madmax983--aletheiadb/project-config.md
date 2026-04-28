---
trigger: always_on
description: **TRUNK IS A PROTECTED BRANCH. YOU MUST ALWAYS USE WORKTREES AND PULL REQUESTS.**
---

# AletheiaDB Architecture & Development Guidelines

## ⚠️ CRITICAL: NEVER COMMIT DIRECTLY TO TRUNK ⚠️

**TRUNK IS A PROTECTED BRANCH. YOU MUST ALWAYS USE WORKTREES AND PULL REQUESTS.**

Before making ANY code changes:
1. Check current branch: `git branch --show-current`
2. If on `trunk`, STOP and create a worktree: `just worktree-new feature/your-feature-name`
3. Work in the worktree, commit there, push, and create a PR
4. NEVER use `git commit` when on trunk - there is a pre-commit hook to prevent this

**The ONLY acceptable commits to trunk are automated merges from approved PRs.**

This is enforced by a pre-commit hook that will block direct commits to trunk.

## Project Overview

AletheiaDB is a high-performance bi-temporal graph database written in Rust. It tracks both **valid time** (when facts were true in reality) and **transaction time** (when facts were recorded in the database), while maintaining performance comparable to regular graph databases for current-state queries.

**Primary Use Case - LLM Integration**: Enable reasoning LLMs to query not just current knowledge, but see how that knowledge evolved over time. This allows LLMs to understand temporal context, track when facts changed, reason about causality, and detect contradictions through provenance tracking.

## Quick Architecture Reference

### Core Principles

1. **Performance First**: Current-state queries <1µs single-hop, temporal queries <10ms reconstruction
2. **Storage Efficiency**: Anchor+delta compression, <2X overhead vs non-temporal storage
3. **Correctness**: ACID guarantees via WAL, MVCC snapshot isolation, crash recovery with checkpoint-based replay

### Hybrid Storage

```
Query Engine
     │
     ├─→ Current Storage (fast path, no temporal overhead)
     └─→ Historical Storage (temporal path, anchor+delta compressed)

Recovery Flow:
   Startup → Load Checkpoint → Replay WAL → Restore Indexes → Ready
```

**Durability & Recovery:**
- **Synchronous Mode**: fsync on every commit, no data loss
- **GroupCommit Mode**: batched fsync with waiting, ACID-compliant, ~100K+/sec
- **Async Mode**: background fsync, eventual consistency, ~500K+/sec
- **Recovery**: Checkpoint-based WAL replay, <5s for 10K nodes/50K edges
- **Checkpoints**: Periodic snapshots for fast startup without full WAL replay

**See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for complete architecture documentation.**

## Rust Coding Standards

**See [docs/CODING_STANDARDS.md](docs/CODING_STANDARDS.md) for comprehensive coding guidelines.**

### Quick Reference

| Guideline | Rule | Why |
|-----------|------|-----|
| **Type Safety** | Use newtype wrappers for IDs (NodeId, EdgeId) | Prevents ID mix-ups |
| **ID Security** | Always validate IDs, keep `new_unchecked()` as `pub(crate)` | Prevents overflow/DoS attacks |
| **Error Handling** | Never `.unwrap()` in production | Prevents panics |
| **Allocations** | Reuse buffers, pre-allocate when size known | Reduces GC pressure |
| **Concurrency** | Lock-free structures for hot paths | Maximizes throughput |
| **Async** | Only for I/O, not CPU-bound work | Avoid overhead |
| **Unsafe** | Document safety invariants with `// SAFETY:` | Required for correctness |

**Critical**: IDs exceeding `MAX_VALID_ID` (u64::MAX - 1000) are rejected to prevent DoS attacks.

## Testing Requirements

**See [TESTING.md](TESTING.md) for detailed testing instructions.**

### Coverage Requirements

- **Minimum 85% line coverage** (current: 86.45%)
- **Minimum 88% function coverage** (current: 89.10%)
- **Minimum 88% region coverage** (current: 88.91%)

### Quick Commands

```bash
just test              # Run all tests
just coverage          # Generate HTML coverage report
just coverage-check    # Verify coverage meets thresholds
just bench             # Run benchmarks (includes recovery benchmarks)
just check-all         # Full quality check (tests, coverage, lint)
```

### Mandatory Benchmarks

All performance-critical features must include benchmarks:

- **Current-state queries**: <1µs single-hop traversal
- **Temporal queries**: <10ms reconstruction
- **WAL operations**: Throughput targets per durability mode
- **Recovery operations**: <5s for medium datasets (10K nodes, 50K edges)
  - Checkpoint creation/loading
  - WAL replay with various entry counts
  - Full crash recovery scenarios
- **Vector search**: <10ms k-NN (k=10, 1M vectors)
- **Hybrid queries**: <30ms graph+vector+temporal

### Miri - Undefined Behavior Detection

All `unsafe` code must be validated with [Miri](https://github.com/rust-lang/miri).

**Quick Commands:**
```bash
just miri-setup        # Install miri (one-time)
just miri              # Run miri on all tests
just miri-test name    # Run specific test
```

**When to run Miri:**
- Before committing any changes to `unsafe` blocks
- After modifying SIMD, lock-free, or concurrent code
- When working with raw pointers, transmute, or FFI

**See [docs/MIRI.md](docs/MIRI.md) for complete guide, configuration, and troubleshooting.**

## Major Features

### Write-Ahead Log (WAL)

Striped lock-free ring buffer architecture for high-throughput concurrent writes.

**Performance:**
- **Synchronous**: ~1.5ms latency, ~600/sec throughput, Full ACID

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madmax983/AletheiaDB](https://github.com/madmax983/AletheiaDB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
