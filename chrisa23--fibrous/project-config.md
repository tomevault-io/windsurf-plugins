---
trigger: always_on
description: Fibrous is a high-performance .NET concurrency library and actor-like concurrency toolbox.
---

# AGENTS.md

## Purpose
Fibrous is a high-performance .NET concurrency library and actor-like concurrency toolbox.

Core concepts in this repository:
- Fibers: ordered execution contexts
- Channels / Ports: messaging conduits and endpoints
- Scheduling: delayed and repeated work
- Messaging patterns: pub/sub, queueing, request/reply, snapshot/state, batching
- Specialized fibers including WPF/UI integration and test/stub fibers

Agents should preserve the library’s main value:
make concurrent systems easier to reason about while remaining practical and performant.

## Current repo context
- Primary target frameworks include .NET Standard 2.0, .NET 8, and .NET 10
- There is a separate .NET Windows project for WPF integration
- The repository is in a v7 era where some API changes may be acceptable, but they should still be deliberate and well-justified
- The repo has strong existing test coverage, though more improvement is welcome
- Benchmark coverage exists but is old and should be treated as potentially stale until refreshed

## Default change philosophy
Prefer:
1. Correctness under concurrency
2. Clear behavioral semantics
3. Backward compatibility where practical
4. Small, reviewable changes
5. Test-backed improvements
6. Performance improvements supported by evidence

Even though v7 may allow API changes, do not make breaking changes casually.
Propose them clearly, explain the benefit, and isolate them from unrelated cleanup.

## Agent operating modes
Unless the task explicitly says otherwise, use one of these modes:

### 1. Analyze only
Read code, tests, docs, and project structure.
Return findings, risks, and suggestions.
Do not edit files.

### 2. Focused implementation
Make a small, well-scoped change with tests.

### 3. Proposal first
When a change is likely to affect public API, ordering, disposal, scheduling, or UI-thread behavior, provide a proposal before editing.

Default to **Analyze only** for first-pass repo exploration tasks.

## Concurrency-specific rules
This repository contains concurrency primitives. Treat changes as high risk when they touch:
- ordering guarantees
- subscriber dispatch behavior
- thread-safe publishing
- request/reply timing or completion semantics
- scheduling behavior
- batching behavior
- disposal and child disposal
- UI thread / dispatcher affinity
- stub fiber determinism
- queue draining or shutdown behavior
- synchronization internals
- fairness / starvation characteristics

When modifying concurrency-sensitive code:
- identify the invariant being preserved
- explain what protects shared state
- call out race, deadlock, reentrancy, and disposal risks
- add or update tests for the affected semantics

## Public API guidance
The repository may accept API improvements for v7, but agents must:
- avoid accidental breaking changes
- clearly identify proposed API breaks
- explain migration impact
- prefer additive APIs when the tradeoff is close
- separate breaking API redesign from internal modernization

Do not rename or remove public APIs unless the task explicitly supports it.

## Modernization guidance
Safe modernization work includes:
- improving nullable annotations
- tightening argument validation
- updating XML docs and examples
- refreshing project/packaging configuration
- internal refactors that preserve semantics
- improving cancellation and disposal handling
- reducing allocations in hot paths
- cleaning up old benchmark infrastructure
- modernizing targeted code paths for .NET 8/9 while preserving the multi-targeted design

Be cautious with:
- introducing async changes that alter ordering
- replacing synchronization primitives
- changing exception behavior
- changing callback timing
- changing scheduler precision or repeat behavior
- altering WPF dispatcher behavior
- using .NET 8/9-only APIs in shared code without proper multi-targeting guards

## Multi-targeting guidance
Because the repo targets .NET Standard 2.0, .NET 8, and .NET 10:
- preserve compatibility in shared code unless a task explicitly targets a newer framework path
- use conditional compilation only when it clearly improves performance, maintainability, or platform support
- avoid introducing framework-specific behavior differences unless documented and tested
- ensure any .NET 8/9 optimizations do not silently change .NET Standard behavior

For WPF integration:
- keep UI-thread affinity explicit
- do not assume behavior from non-WPF fibers applies to UI fibers
- isolate Windows-specific logic cleanly

## Performance guidance
Fibrous is performance-oriented, but not at the expense of correctness or clarity.
Agents should:
- avoid unnecessary allocations in hot paths
- avoid LINQ and closure-heavy code in hot paths when measurable
- prefer evidence-backed performance claims
- refresh or extend benchmarks when changing important internals
- treat old benchmarks as useful hints, not final truth

Do not make speculative micro-optimizations that reduce readability without proof.

## Testing expectations
For non-trivial changes, add or update tests.

Important areas to cover:
- fiber ordering
- enqueue semantics
- thread-safe publishing
- single vs multiple subscriber behavior
- request/reply behavior
- scheduling and repeat scheduling
- batching

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chrisa23/Fibrous](https://github.com/chrisa23/Fibrous) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
