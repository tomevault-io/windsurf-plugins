---
trigger: always_on
description: Instructions for AI agents and human developers working on Bloodhound.
---

# Bloodhound Development Guide

Instructions for AI agents and human developers working on Bloodhound.

## Project Overview

Bloodhound is a **deterministic simulation testing platform** for hunting bugs in distributed systems using a modified QEMU hypervisor.

**Key Goals:**
- Perfect reproducibility: Same seed = identical execution, always
- Language agnostic: Test any containerized application without modification
- Systematic fault injection: BUGGIFY-style deterministic faults
- Time-travel debugging: Full replay capability with GDB integration

## Engineering Philosophy

**Three pillars ground all code in Bloodhound:**

1. **[TigerStyle](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/TIGER_STYLE.md)** - Coding standards (assertions, explicit limits, naming)
2. **Formal Specifications** (`specs/tla/`, `src/stateright/`) - Protocol correctness
3. **Deterministic Simulation Testing** - Fault tolerance verification

**Before writing code, consult all three.** They define what "correct" means.

Priority order: **Safety > Performance > Developer Experience**

### The Verification Pyramid

All three pillars are **connected through shared invariants** (`src/invariants/`):

```
         TLA+ Specs (specs/tla/*.tla)
                    │ mirrors
         Shared Invariants (src/invariants/*.rs)  ← SINGLE SOURCE OF TRUTH
                    │ used by
    ┌───────────────┼───────────────┐
    ▼               ▼               ▼
Stateright      DST Tests      Production Code
(exhaustive)    (simulation)   (TigerStyle)
```

**Each invariant has:**
- `name` - matches TLA+ property name
- `tla_spec` - source spec file reference
- `tla_line` - line number in spec
- `holds` / `violation` - result with context

**Example:** When implementing snapshot tree operations:
1. Read `specs/tla/SnapshotTree.tla` for the protocol
2. Use `SnapshotPropertyChecker` from `invariants/snapshot.rs` in your tests
3. Properties like `no_orphan_snapshots` are verified identically across all layers

### Core Principles

1. **Determinism Above All**: Every execution with the same seed must produce identical results. Non-determinism is a bug.

2. **Simulation-First**: Test infrastructure comes before implementation. If you can't test it deterministically, don't build it.

3. **Fault Injection is Not Optional**: Distributed systems fail in complex ways. We inject faults systematically, not as an afterthought.

4. **Honest Documentation**: Document what works AND what doesn't. Never claim something is tested if it isn't.

## Quick Rules

### Assertions (2+ per function)
```rust
use crate::tigerstyle::{assert_precondition, assert_postcondition, assert_invariant};

fn transfer(from: &mut Account, to: &mut Account, amount: u64) -> Result<()> {
    // Preconditions: What must be true before
    assert_precondition!(amount > 0, "amount must be positive");
    assert_precondition!(from.balance >= amount, "insufficient balance");

    let old_from_balance = from.balance;
    from.balance -= amount;
    to.balance += amount;

    // Postconditions: What must be true after
    assert_postcondition!(from.balance < old_from_balance, "from balance must decrease");

    Ok(())
}
```

### Explicit Limits (bound everything)
```rust
const SNAPSHOT_SIZE_BYTES_MAX: usize = 1024 * 1024 * 1024;  // 1GB
const VM_COUNT_MAX: usize = 64;
const SIMULATION_STEPS_MAX: u64 = 1_000_000;
const FAULT_PROBABILITY_MAX: f64 = 0.5;

if snapshot_size > SNAPSHOT_SIZE_BYTES_MAX {
    return Err(Error::LimitExceeded(...));
}
```

### Big-Endian Naming (most significant first)
```rust
// GOOD
snapshot_size_bytes_max
simulation_time_ms
fault_probability_default

// BAD
max_snapshot_size
simulationTimeMs
```

### Deterministic RNG Usage
```rust
// GOOD: Use FaultActor's seeded RNG
let should_fault = fault_actor.maybe_fault("disk_write", 0.01);

// BAD: Non-deterministic random
let should_fault = rand::random::<f64>() < 0.01;
```

### Error Handling
```rust
// GOOD: Typed errors with context
return Err(SimulationError::VmFailed {
    vm_id,
    reason: "hypercall timeout".into(),
    seed: self.seed,
});

// BAD: String errors
return Err("VM failed".into());
```

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                       BLOODHOUND                            │
│              (Modified QEMU Hypervisor)                     │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Virtual Time │  │ Fault Inject │  │ State Snap   │      │
│  │ (TSC, HPET)  │  │ (Net, Disk)  │  │ (CoW, Tree)  │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
├─────────────────────────────────────────────────────────────┤
│                    GUEST VMs (Containers)                   │
│  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐           │
│  │  web   │  │ redis  │  │postgres│  │  ...   │           │
│  └────────┘  └────────┘  └────────┘  └────────┘           │
└─────────────────────────────────────────────────────────────┘
```

### Actor Hierarchy

```
SimulationCoordinator
├── TimeActor (owns virtual clock, deterministic time)
├── FaultActor (schedules/injects faults deterministically)
├── EventCollectorActor (trace recording)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nerdsane/bloodhound](https://github.com/nerdsane/bloodhound) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
