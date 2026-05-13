---
trigger: always_on
description: Research repo exploring whether vanilla transformer primitives can implement a working computer (stack machine). Inspired by Percepta's "Can LLMs Be Computers?" blog post (Mar 2026).
---

# LLM-as-Computer

Research repo exploring whether vanilla transformer primitives can implement a working computer (stack machine). Inspired by Percepta's "Can LLMs Be Computers?" blog post (Mar 2026).

## Muninn Boot

This repository is developed by Oskar Austegard using Claude sessions sharing persistent memory via Muninn. Boot loads profile, operational context, and prior findings into the session.

**Boot is automatic.** The SessionStart hook (`.claude/hooks/session-start.sh`) runs `boot()` at the beginning of every Claude Code on the web session.

Credentials auto-detect from environment or well-known paths (`/mnt/project/turso.env`, `/mnt/project/muninn.env`, `~/.muninn/.env`). If boot fails, the hook logs a warning and continues.

### Decision Traces

After completing meaningful work (new phase, training run, key finding), store a memory:

```python
remember(
    "Phase N result: [what was found]. Key insight: [what it means]. "
    "Constraint: [if any]. Next: [what follows].",
    "analysis",
    tags=["LLM", "architecture", "research", "phase-N"],
    priority=1
)
```

Lead with *why*, not *what* — the diff shows what. Include surprises, rejected approaches, and architectural implications.

### Regular Memory Updates

Store progress updates to Muninn **regularly during work**, not just at the end. Any time you complete a meaningful sub-task, hit a blocker, or discover something unexpected, call `remember()` so future sessions have context. Aim for at least one memory store per significant milestone (e.g., "tests passing", "new opcode working", "design decision made"). This protects against session cutoffs losing context.

## Project Context

### What This Is

A bottom-up validation of whether transformer attention + FF layers can implement program execution. Each phase isolates a primitive, tests it numerically, then composes with prior phases.

### Key Architectural Insight

**Attention is lookup; feed-forward is routing.** Attention is cheap and reliable (pattern matching, memory addressing). FF layers must learn crisp categorical decisions (opcode dispatch) — this is the hard part. Width > depth for learning execution.

### Parabolic Encoding

The workhorse primitive: `k = (2j, -j²)` encodes position j such that dot-product attention peaks sharply at the target position. Same encoding addresses both program memory and stack memory without interference. Phase 2b extended this past float32 limits via residual (bit-split) addressing.

## Phases

| Phase | File | Status | What It Proves |
|-------|------|--------|----------------|
| 1 | phase1_hull_cache.py | Complete | O(log t) lookup via ternary search on parabolic keys |
| 2 | phase2_parabolic.py | Complete | Parabolic encoding as exact memory addressing |
| 2b | phase2b_address_limits.py | Complete | Residual addressing scales to 25M+ range |
| 3 | phase3_cumsum.py | Complete | Cumulative sum tracks instruction pointer / stack pointer |
| 4 | phase4_stack_machine.py | Complete | Hand-wired transformer executes PUSH/POP/ADD/DUP/HALT correctly |
| 5 | phase5_training.py | Complete | Tiny model learns execution grammar (56% acc) but not perfect traces |
| 6 | phase6_curriculum.py | Complete | Curriculum learning: 56%→85% acc, 0→39/50 perfect traces |
| 7 | phase7_percepta_arch.py | Complete | Percepta architecture (d=36,h=18,L=7): 84.6% acc, DIFF+ADD still 0% |
| 8 | phase8_microop_traces.py | Complete | Micro-op decomposition proves retrieval is solved; arithmetic is bottleneck |
| 9 | phase9_weighted_arithmetic.py | Complete | Weighted loss perfects doubling (100%) but DIFF+ADD stays 0% |
| 11 | phase11_compile_executor.py | Complete | Compiled execution: correct traces, extended ISA (SUB/JZ/JNZ), O(log t) path |
| 12 | phase12_percepta_model.py | Complete | Full PyTorch compiled transformer with real nn.Linear weight matrices |
| 13 | phase13_isa_completeness.py | Complete | ISA completeness: SWAP/OVER/ROT + Fibonacci, multiply, power-of-2, sum, parity |

### Phase 5 Key Finding

Wide model (d=64, heads=4, layers=2, 137K params) reaches 56% token accuracy (vs 0.5% chance) but 0/50 perfect traces. The model learns *structure* but not *precise routing*. This is the attention-vs-FF gap made concrete: good at finding operands, bad at dispatching operations.

### Phase 6 Key Findings

Curriculum learning confirms the hypothesis: staged instruction complexity (PUSH-only → +POP/DUP → full set) improves accuracy from 56% to 85% with 39/50 perfect traces.

**Three deep diagnostics revealed the bottleneck progression:**
1. **Copy bottleneck (solved):** The model couldn't copy values from program memory. Fix: more data (5K samples) — the copy mechanism IS learnable, just data-starved at 1K.
2. **Non-arithmetic execution (solved):** Stage 2 (PUSH/POP/DUP) achieves 50/50 perfect traces. The model fully learns stack operations that don't require arithmetic.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oaustegard/llm-as-computer](https://github.com/oaustegard/llm-as-computer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
