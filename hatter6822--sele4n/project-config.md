---
trigger: always_on
description: seLe4n is a production-oriented microkernel written in Lean 4 with machine-checked
---

# CLAUDE.md — seLe4n project guidance

## What this project is

seLe4n is a production-oriented microkernel written in Lean 4 with machine-checked
proofs, improving on seL4 architecture. Every kernel transition is an executable
pure function with zero `sorry`/`axiom`. First hardware target: Raspberry Pi 5.
Lean 4.28.0 toolchain, Lake build system, version 0.28.0.

## Build and run

```bash
# Environment setup (runs automatically via SessionStart hook — no build)
./scripts/setup_lean_env.sh --skip-test-deps

# Full setup including test dependencies (shellcheck, ripgrep)
./scripts/setup_lean_env.sh

# Manual build (run separately after setup)
source ~/.elan/env && lake build

# Run executable trace harness
lake exe sele4n
```

## Validation commands (tiered)

```bash
./scripts/test_fast.sh      # Tier 0+1: hygiene + build
./scripts/test_smoke.sh     # Tier 0-2: + trace + negative-state
./scripts/test_full.sh      # Tier 0-3: + invariant surface anchors
NIGHTLY_ENABLE_EXPERIMENTAL=1 ./scripts/test_nightly.sh  # Tier 0-4
```

Run at least `test_smoke.sh` before any PR. Run `test_full.sh` when changing
theorems, invariants, or documentation anchors.

## Module build verification (mandatory)

**Before committing any `.lean` file**, you MUST verify that the specific
module compiles:

```bash
source ~/.elan/env && lake build <Module.Path>
```

For example, if you modified `SeLe4n/Kernel/RobinHood/Bridge.lean`:
```bash
lake build SeLe4n.Kernel.RobinHood.Bridge
```

**`lake build` (default target) is NOT sufficient.** The default target only
builds modules reachable from `Main.lean` and test executables. Modules that
are not yet imported by the main kernel (e.g., `RobinHood` before N4
integration) will silently pass `lake build` even with broken proofs.

A pre-commit hook enforces this automatically. Install it:
```bash
cp scripts/pre-commit-lean-build.sh .git/hooks/pre-commit
```

The hook:
1. Detects staged `.lean` files
2. Builds each modified module via `lake build <Module.Path>`
3. Checks for `sorry` in staged content
4. **Blocks the commit** if any build fails or sorry is found

Do NOT bypass the hook with `--no-verify`.

## Source layout

```
SeLe4n/Prelude.lean              Typed identifiers, monad foundations
SeLe4n/Machine.lean              Machine state primitives
SeLe4n/Model/Object.lean         Kernel objects (re-export hub)
  Object/Types.lean              Core data types, TCB, Endpoint, Notification
  Object/Structures.lean         VSpaceRoot, CNode, KernelObject, CDT helpers
SeLe4n/Model/State.lean          Kernel/system state representation
SeLe4n/Model/IntermediateState.lean  Q3-A: Builder-phase state with invariant witnesses
SeLe4n/Model/Builder.lean        Q3-B: Builder operations (invariant-preserving state construction)
SeLe4n/Model/FrozenState.lean    Q5: FrozenMap, FrozenSet, FrozenSystemState, freeze function
SeLe4n/Model/FreezeProofs.lean   Q6: Freeze correctness proofs (lookup equiv, radix equiv, invariant transfer)
SeLe4n/Kernel/Scheduler/*        Scheduler transitions + invariants
  Operations.lean                Re-export hub
    Operations/Selection.lean    EDF predicates, thread selection
    Operations/Core.lean         Core transitions (schedule, handleYield, timerTick)
    Operations/Preservation.lean Scheduler invariant preservation theorems
  PriorityInheritance.lean       Re-export hub (D4)
    PriorityInheritance/BlockingGraph.lean   Blocking relation, chain walk, acyclicity, depth bound
    PriorityInheritance/Compute.lean         computeMaxWaiterPriority
    PriorityInheritance/Propagate.lean       updatePipBoost, propagate/revert priority inheritance
    PriorityInheritance/Preservation.lean    Frame lemmas (scheduler, IPC, cross-subsystem)
    PriorityInheritance/BoundedInversion.lean Parametric WCRT bound, determinism
  Liveness.lean                  Re-export hub (D5)
    Liveness/TraceModel.lean     Trace model types, query predicates, counting functions
    Liveness/TimerTick.lean      Timer-tick budget monotonicity, preemption bounds
    Liveness/Replenishment.lean  CBS replenishment timing bounds
    Liveness/Yield.lean          Yield/rotation semantics, FIFO progress bounds
    Liveness/BandExhaustion.lean Priority-band exhaustion analysis
    Liveness/DomainRotation.lean Domain rotation bounds
    Liveness/WCRT.lean           WCRT hypotheses, main theorem, PIP enhancement
SeLe4n/Kernel/Capability/*       CSpace/capability ops + invariants
  Invariant.lean                 Re-export hub
    Invariant/Defs.lean          Core invariant definitions, transfer theorems
    Invariant/Authority.lean     Authority reduction, badge routing
    Invariant/Preservation.lean  Operation preservation, lifecycle integration
SeLe4n/Kernel/IPC/*              IPC subsystem
  Operations.lean                Re-export hub
    Operations/Endpoint.lean     Core endpoint/notification ops
    Operations/CapTransfer.lean  IPC capability transfer (WS-M3)
    Operations/Timeout.lean      Z6 timeout-driven IPC unblocking
    Operations/Donation.lean     Z7: SchedContext donation wrappers + preservation proofs
    Operations/SchedulerLemmas.lean Scheduler preservation lemmas
  DualQueue.lean                 Re-export hub

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hatter6822) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
