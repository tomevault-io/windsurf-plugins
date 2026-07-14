---
trigger: always_on
description: Build IX v2 as a simple, clean, capability-complete Rust search engine and benchmark platform. Simplicity is for tidiness and slop avoidance, never for capability loss. Primary performance objective: become faster than ripgrep and other top search competitors (for example fff and codedb) on transparent benchmark suites, with a tracked target of at least 50% faster versus ripgrep on agreed workloads.
---

# AGENTS.md

## Mission
Build IX v2 as a simple, clean, capability-complete Rust search engine and benchmark platform. Simplicity is for tidiness and slop avoidance, never for capability loss. Primary performance objective: become faster than ripgrep and other top search competitors (for example fff and codedb) on transparent benchmark suites, with a tracked target of at least 50% faster versus ripgrep on agreed workloads.

## Core Patterns (Required)
- `ATOMIC`: ship one coherent slice at a time.
- `DRY`: no duplicated ownership logic.
- `STEP`: deterministic sequence, no random side quests.
- `SOLID`: stable contracts and clear module boundaries.
- `LEVER`: maximize impact with minimum durable architecture change.
- `YAGNI`: no speculative systems without present value.
- `MODULAR`: narrow ownership per module.
- `REUSE`: prefer proven patterns and reusable components.
- `UNIFIRM`: consistent naming, contracts, and behavior.
- `PARITY`: CLI, stats, tests, and docs stay aligned.

## Anti-Patterns (Forbidden)
- Workarounds.
- Assumption-driven changes without direct evidence.
- File sprawl.
- Parallel systems for the same responsibility.
- Prompt scaffolding leakage into user-visible output.
- Non-problem solving.
- Generic or vague implementation notes.
- Fallback behavior that hides unsupported capability.

## Operating Principles
- Observe > Reflect > Make.
- Keep focus on the active objective.
- Maintain strict directory depth hygiene with self-explanatory naming.
- Favor compile-time boundaries over runtime shims.
- Minimum change means minimum durable architecture change, not minimum typing.
- When a performance regression resists local explanations, zoom out from the immediate branch or gate and re-price the whole workload shape before editing more control flow. Check retained bytes, slowest files, and tail-dominant surfaces first so the fix targets the real bottleneck instead of the loudest symptom.
- Prefer narrow repairs at the dominant cost center over another broad scheduler toggle. If a few giant files are setting the tail, fix scan-kernel or ownership behavior there first, then revisit higher-level traversal doctrine from the lower tail-tax floor.
- Native predecessor parity is a hard performance contract. A suite split near parity, such as 6 wins and 6 losses against the installed previous IX binary, means the investigation is not done even when individual wins are impressive. Use the previous/native binary as the floor to beat, not as a loose comparison target.
- Treat benchmark glare as hostile. Before editing, dive below headline ratios into byte-level workload shape, retained byte volume, candidate filters, regex decomposition, byte charting, match density, thread topology, shard geometry, aggregation cost, and slowest-file tails. The target is the actual cost center, not the most visible branch.
- Never promote assumption-first optimization. When a candidate idea is still unproven, isolate its core mechanism in the smallest removable slice that can be benchmarked against the current canonical binary and the installed native binary. Expand only after repeated interleaved samples show a real average win with match-count parity.
- Research must be specific and demand-driven. If byte charting, regex execution, SIMD/memmem behavior, Aho-Corasick strategy, thread scheduling, cache locality, or Windows filesystem behavior becomes the active unknown, use focused web/Insect research on that exact mechanism before designing the durable fix. Do not perform broad topic surveys as a substitute for profiling the live lane.
- Prefer bottom-up proof over broad rewrites. A quick isolated function, temporary probe, microbench, or telemetry counter is acceptable when it narrows the blast radius and can be removed cleanly. A full implementation without a preceding proof slice is not acceptable for performance work.
- Require enough samples to defeat machine-load noise. One or two faster pairs are not proof. Use interleaved multi-pair comparisons, compare medians and win counts, and rerun suspicious near-ties before calling a lane fixed or lost.
- Keep the 50% faster ambition visible. If improvements flatten, do not keep shaving the same surface. Re-examine the architecture for a qualitatively different path: less scanning, better prefilter rejection, cheaper aggregation, reduced materialization, sharper prepared-input reuse, or different concurrency ownership.
- Once native install is present, prefer `ix` for local search and search-validation workflows. During command-surface migration, repo-local `target/release/ix.exe` is the preferred binary and `target/release/iex-cli.exe` is compatibility-only. Do not use `rg` for local repo search in this workspace unless `ix` is unavailable and that blocker is recorded in evidence.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [savageops/ix-rust](https://github.com/savageops/ix-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
