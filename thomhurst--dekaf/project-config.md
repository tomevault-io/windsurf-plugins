---
trigger: always_on
description: Dekaf is a high-performance, pure C# Apache Kafka client. Performance is the product: improve throughput, latency, CPU, and allocations together.
---

# Dekaf Development Guide

Dekaf is a high-performance, pure C# Apache Kafka client. Performance is the product: improve throughput, latency, CPU, and allocations together.

## Performance requirements

- Protected metrics are throughput, p50/p99 latency, CPU per message, allocations per message, and stability. Do not trade one for another without explicit maintainer approval, including for refactoring or review feedback.
- Hot paths include serialization, batch append/drain, channel writes, per-message produce/consume, and receive/parse loops. Keep per-message fast paths synchronous with `ValueTask`, allocate 0 B per message, and amortize necessary work per batch, connection, or epoch. Say in the PR which costs are per message and which are amortized.
- Hot paths must avoid LINQ, capturing lambdas, uncached delegates, boxing, string formatting, allocating collection conversions/iterators, interface-enumerator allocations, per-message async state machines, `Task.Run`, thread-pool hops, locks, exceptions as control flow, and O(n) scans or cleanup loops. Prefer spans, pooled buffers, static callbacks with explicit state, channels, and `Interlocked`. Remove completed operations from tracking collections and coordinate disposal of in-flight work, observing failures.
- A hot-path change needs a steady-state `[MemoryDiagnoser]` fixture in `tools/Dekaf.Benchmarks/Benchmarks/Unit` that exercises the changed code. Add or extend the fixture in the same PR as the product change; the gate builds the PR's fixture source against both revisions, so nothing has to land on `main` first.

## Performance gate

Every PR that changes `src/`, `tools/Dekaf.Benchmarks/` or the build inputs runs the [performance gate](.github/workflows/performance-gate.yml). It selects benchmark classes with `.github/scripts/performance_gate.py` (four hot-path sentinels for any product change, a few classes per hot component directory, plus every fixture class the PR touches) and runs one `ubuntu-latest` job per class. Each job measures baseline, candidate, baseline (A1/B/A2) back to back on one VM and compares medians. A class finishes in about 10 to 25 minutes, or up to about 50 when a regression is re-measured.

- `REGRESSION` fails the check only when the candidate is slower than both controls beyond the tolerance for that case size (20% under 100 ns, 15% under 1 µs, 10% from 1 µs), or allocates at least 24 B/op and 1% more than both controls, and the loss reproduces on an immediate repeat of those cases. Everything else passes; control drift and one-sided differences are notes. When the PR fixtures do not compile against the baseline, cases from fixture files the PR changed are reported as `NOT COMPARED`, and a class the PR adds is measured on the candidate alone.
- Hosted runners drift 10% to 30% between processes on nanosecond cases even for identical binaries. A single delta inside the tolerance is noise, not a win or a loss. Local measurements are diagnostic only.
- On a red gate, open the job summary table and download the artifact (original BenchmarkDotNet exports and logs), then fix the product or state the tradeoff in the PR for maintainer approval. Do not rerun until green, widen tolerances, add PR-specific rules to `performance_gate.py`, or split fixtures into a separate PR. A `PASS` needs no rerun.
- Fixtures the gate runs must be steady-state: no `[IterationSetup]`/`[IterationCleanup]`, `InvocationCount` or cold-start strategies (those are skipped with a reason), and under about 16 expanded cases per class.
- CI runs the benchmarks. Build a new or changed fixture before pushing, but do not run benchmark suites locally for evidence; read the gate's job summary instead.
- CI also runs the stress tests. Do not dispatch stress-lane runs yourself, and do not block a PR on a manual stress run.
- Do not commit benchmark outputs, logs, reports, traces or evidence Markdown. Job summaries and artifacts are the evidence store; PR comments link them. The repository artifact-policy check enforces this.

## Library conventions and contracts

- Use nullable reference types, init-only options, fluent builders, and existing modern C# conventions. Public APIs expose interfaces; implementations are internal or sealed.
- All awaits in `src/` use `ConfigureAwait(false)`; tests do not require it. Do not block on tasks with `.Result` or `.Wait()`.
- Kafka-specific exceptions derive from `KafkaException`; use `IsRetriable` for retry decisions.
- Producer `BufferMemory` limits apply to every append path, including the arena fast path. Exhaustion backpressures `ProduceAsync` until space is available.
- `ProduceAsync` cancellation before append prevents delivery, including during metadata lookup, channel writes, and memory reservation. After append, cancellation stops the caller's wait while delivery continues.
- `FlushAsync` cancellation stops waiting while batches continue sending. `FireAsync` has no cancellation-token overload; use `FlushAsync(cancellationToken)` for cancellable delivery waiting.

## Build and test


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thomhurst/Dekaf](https://github.com/thomhurst/Dekaf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
