---
trigger: always_on
description: Before writing any code, read these three documents — they are the source of truth:
---

# Claude Code Guidelines for redis-rust

## ⚠️ MANDATORY: Read These First

Before writing any code, read these three documents — they are the source of truth:

1. **[docs/HARNESS.md](docs/HARNESS.md)** — 4-layer verification loop, expected outputs, Tcl harness pitfalls, command addition checklist
2. **[docs/RUST_STYLE.md](docs/RUST_STYLE.md)** — Rust coding standards: error handling, file size limits, clone avoidance, iterator patterns, assertion requirements, checked arithmetic
3. **[docs/DST_GUIDE.md](docs/DST_GUIDE.md)** — Deterministic simulation testing methodology: SimulatedRng, VirtualTime, buggify fault injection, shadow state comparison

Violations of RUST_STYLE.md or DST_GUIDE.md will be caught by code review and CI. Do not skip them.

### On-Demand Skills

Invoke these skills to inject domain expertise before working on specific areas:

| Skill | When to use |
|-------|-------------|
| `/distributed-systems` | Before working on replication, gossip, CRDTs, or multi-node code |
| `/actor-model` | Before working on sharded_actor, connection handler, or message passing |
| `/dst` | Before writing or modifying DST tests, fault injection, or shadow state |
| `/tigerstyle` | Before modifying executor or data structure code (assertions, checked arithmetic) |
| `/rust-dev` | Before writing any Rust code in this project (style, conventions, patterns) |
| `/formal-verification` | Before working on TLA+ specs, Stateright models, Kani proofs, or Maelstrom |

Skills are defined in `.claude/agents/` and are invocable via `/skill-name` in Claude Code.

## ⚠️ MANDATORY: Default Development Workflow

**For EVERY code change, follow this order:**

### 1. Plan First (Architecture)
Before writing any code:
- Identify which components are affected
- Check if new I/O abstractions are needed for testability
- Determine if actor boundaries need modification
- Document the approach in a brief plan

### 2. TigerStyle First
Every function/method must have:
- `debug_assert!` for preconditions at entry
- `debug_assert!` for postconditions/invariants after mutations
- Checked arithmetic (use `checked_add`, `checked_sub`, etc.)
- Explicit error handling (no `.unwrap()` in production paths)
- Early returns for error cases

### 3. DST-Compatible Design
Every new component must be:
- **Simulatable**: All I/O through trait abstractions
- **Deterministic**: No hidden randomness, use `SimulatedRng`
- **Time-controllable**: Use `VirtualTime`, not `std::time`

### 4. Write Tests Before/With Code
- Unit tests for pure logic
- DST tests with fault injection for I/O components
- Multiple seeds (minimum 10) for simulation tests
- Redis equivalence tests for command behavior changes
- Use Zipfian distribution (not uniform) for realistic workload tests

### 5. Design-by-Contract (TigerStyle Assertions)
- State invariants must be checkable at any point
- Add `verify_invariants()` methods to stateful structs
- Run invariant checks in debug builds after every mutation

Note: VOPR (Viewstamped Operation Replicator) is TigerBeetle's specific DST tool for testing their Viewstamped Replication consensus protocol — it is NOT a synonym for DST in general. Our DST methodology follows the FoundationDB approach (seed-based determinism, simulated I/O, fault injection). The `verify_invariants()` pattern comes from TigerStyle / Design-by-Contract, not VOPR.

---

## Project Philosophy

This project follows **Simulation-First Development** inspired by FoundationDB and TigerBeetle. The core principle: **if you can't simulate it, you can't test it properly**.

## Architecture Principles

### 1. Deterministic Simulation Testing (DST)

All I/O operations must go through abstractions that can be:
- **Simulated**: Deterministic, controllable behavior
- **Fault-injected**: Network partitions, disk failures, message drops
- **Time-controlled**: Fast-forward time, test timeout scenarios

```rust
// GOOD: I/O through trait abstraction
trait ObjectStore: Send + Sync {
    async fn put(&self, key: &str, data: &[u8]) -> Result<()>;
    async fn get(&self, key: &str) -> Result<Vec<u8>>;
}

// BAD: Direct I/O that can't be simulated
std::fs::write(path, data)?;
```

### 2. Actor Architecture

Components communicate via message passing, not shared mutable state:

```rust
// GOOD: Actor owns state exclusively
struct PersistenceActor {
    state: PersistenceState,  // Owned, not shared
    rx: mpsc::Receiver<Message>,
}

// BAD: Shared state with mutex
struct SharedPersistence {
    state: Arc<Mutex<PersistenceState>>,  // Contention, hard to test
}
```

### 3. TigerStyle Coding

**Assertions (REQUIRED for every mutation):**
```rust
// GOOD: Assert preconditions and postconditions
fn hincrby(&mut self, field: &str, increment: i64) -> Result<i64> {
    debug_assert!(!field.is_empty(), "Precondition: field must not be empty");

    let new_value = self.value.checked_add(increment)
        .ok_or_else(|| Error::Overflow)?;

    self.value = new_value;

    debug_assert_eq!(self.get(field), Some(new_value),
        "Postcondition: value must equal computed result");

    Ok(new_value)
}

// BAD: No assertions, silent failures
fn hincrby(&mut self, field: &str, increment: i64) -> i64 {
    self.value += increment;  // Can overflow!
    self.value

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nerdsane/redis-rust](https://github.com/nerdsane/redis-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
