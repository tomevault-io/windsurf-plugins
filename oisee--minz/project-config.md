---
trigger: always_on
description: Enables zero-overhead widening/narrowing arithmetic via operator syntax.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with the MinZ compiler repository.

---

## 🚨 CURRENT PRIORITIES

### 1. Register Allocator & Loop Codegen Bugs
**Status:** Known, not yet fixed. Blocks complex programs.
- While/for loops: register allocator overwrites operands (same phys reg for two live virtuals)
- `loadToHL` uses stale HL values in multi-expression contexts
- Loop rerolling too aggressive across function call boundaries
- See `docs/adr/` for ADR-0006, ADR-0007

### 2. Iterator Chain Fusion
**Status:** Pipeline correct (11/11 E2E), fusion optimizer live. ~5x perf overhead from memory-backed registers.
- 11/11 E2E hex-verified: forEach, take, skip, map, filter, lambda map/filter, multi-stage chains
- Fusion optimizer inlines small callbacks into DJNZ loops (eliminates CALL/RET)
- **Broken on Z80:** enumerate (B=counter+index conflict), reduce (A overwritten by 2nd SMC param)
- **Bottleneck:** Register allocator puts all virtuals through $F0xx memory (~207T actual vs ~43T ideal per element)
- 87+ tests across 7 layers
- See [Iterator Implementation Status](docs/Iterator_Implementation_Status.md)

### 3. MIR2 Open Bugs
**Status:** 8 tracked bugs (4 fixed, 4 open — 1 blocking 🔴). See **[docs/Open_Bugs_RCA.md](docs/Open_Bugs_RCA.md)** for full RCA.
- 🔴 **BUG-008** Arena codegen: impossible `LD IXL, (IX+d)` + self-pointer loss (blocks struct methods)
- 🟡 **BUG-001** GCD parallel-copy bloat + `$0000` ROM spills (PBQP affinity / spill relocation)
- 🟡 **BUG-006** Zero-size struct globals not emitted (undefined symbol at link time)
- 🟡 **BUG-007** Spurious adapter LD when caller/callee share PFCCO convention
- ✅ **BUG-002** forEach constant rematerialization — fixed 2026-03-12
- ✅ **BUG-003** `ptr[i]` in while loop — fixed 2026-03-12
- ✅ **BUG-004** Non-zero-lo LUT pipeline ordering — fixed 2026-03-12
- ✅ **BUG-005** `applySubSwapNeg` u16 guard — fixed 2026-03-12

### 4. LIR Backend (Guided PBQP+WFC Register Allocation)
**Status:** 🚧 Production-matching codegen for leaf functions, 94.6% C89 corpus convergence.
- **Branch:** `feat/lir-backend` (24+ commits, ~7500 LOC)
- **Pipeline:** MIR2 → Bridge → Combine(ISLE) → isel(PatternTable) → WFC(PBQP-guided) → peephole → emit
- **PBQP→WFC synthesis:** PBQP provides global allocation hints, WFC enforces Z80-specific constraints. Output matches production codegen for leaf functions.
- **Call support:** OpCall lowering with arg setup moves, `DstAllowed` class constraints, tail call opt (CALL+RET → JP, saves 17T)
- **IXH/IXL L2 spill:** Undocumented IX/IY half-registers as call-safe storage (8T). 4 bytes of fast spill without touching stack. No existing Z80 compiler uses this.
- **WFC passes:** forward, backward, vregConsistency, clobberPass (call-safe narrowing), Collapse with `pickPreferred(hints)`
- **Save-before-overwrite:** Bridge-level insertion of save moves for vregs at risk from destructive ALU ops (Z80 accumulator architecture)
- **Peephole:** LD r,r no-op elimination, tail call CALL+RET→JP
- **ISLE combining:** load16_le fusion (FatFS ld_word: 8→2 ops), MUL strength reduction
- **WFC Dimension 2:** Inter-block constraint propagation across CFG edges — `ProgWFC` with RPO collapse, back-edge fixpoint
- **Z80 descriptor:** 22 locs (GPR + IX/IY halves + spill), 41+ patterns, DD/FD prefix rules
- **Corpus:** **948/948 pipeline completion** (100%); VM-verified: **97.9%** C89 on risc32 (15 struct/fptr divergences)
- **Runtime:** `__mul8` (A×B→A, ~80T), `__mul16` (HL×DE→HL, ~200T) — shared routines, emitted once per module
- **Remaining:** EXX shadow regs (L3), ISLE const-MUL reduction, production switch as default `--lir`
- See [Architecture](docs/LIR_Backend_Architecture.md), [Reference](docs/LIR_Backend_Reference.md), [Report 094](reports/2026-03-18-094-LIR-100-Percent-C89-Corpus.md), [ADR-0033](docs/adr/0033-lir-pipeline-integration.md)

### 5. VIR — retired as a backend, kept as an offline oracle
**Status:** ⏸️ **Removed from the compiler on 2026-08-21.** See
[ADR-0043](docs/adr/0043-vir-demoted-to-offline-oracle.md).

The `--vir` flag no longer exists and the pipeline no longer calls the Z3 solver. VIR was the
default until it was measured against the path it replaced:

| Corpus | PBQP (production) | VIR |
|---|---|---|
| `examples/c89`, Z80 asserts on the emulator | **37 pass / 2 fail** | 33 / 6 |
| `examples/abap`, assembles | **28 / 30** | **0 / 30** |

Our own `TestVIR_Assert_GCD` fails — `gcd(12,8)` returns 0 — and nobody saw it because `pkg/vir`
takes ~25 minutes while `go test` gives up at 10. This also matches the April decision recorded in
`reports/2026-04-08-Session-Report-EN.md`: *"Z3 parked as `--vir` flag, PBQP stays production
default."* `DefaultOptions` honoured that; the CLI flag did not.

**What survives.** `pkg/vir` keeps the machine description, the regalloc tables, the GPU mul/div
tables and the peephole rules — all imported by `cmd/mzv`, `cmd/mir2asm` and `cmd/gpu-bench`, none
of which want an SMT solver. The solver itself is now `cmd/vir-oracle`, a research tool that
reports what an optimal allocation *would* be so the precomputed tables have something independent
to be checked against. It needs `z3` on PATH; the compiler needs nothing.

```bash
make vir-oracle

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oisee/minz](https://github.com/oisee/minz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
