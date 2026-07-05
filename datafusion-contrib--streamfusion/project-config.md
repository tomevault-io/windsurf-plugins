---
trigger: always_on
description: This project attempts to run Apache Flink using Apache DataFusion + rust operators via JNI to accelerate stream
---

This project attempts to run Apache Flink using Apache DataFusion + rust operators via JNI to accelerate stream
processing throughput. It is a very similar project to DataFusion comet in spirit, but with stream processing.

The canonical home of this repo is https://github.com/datafusion-contrib/StreamFusion (the `upstream` remote,
branch `main`) — push every commit there. The jordepic GitHub fork (`origin`) is a personal mirror, kept in sync
but never the destination of record. This overrides any general rule about committing OSS work to the fork.

We are aiming for IDENTICAL results in stream processing jobs to flink. To start, we'll focus on Flink SQL. As column
oriented streaming sources pop up more and more (see Fluss, for example, or open table CDC), being able to run jobs
at massive throughput becomes more important.

The one place identical results are impossible is an inherently non-deterministic function — `PROCTIME()` /
processing time, `NOW()`/`CURRENT_TIMESTAMP`, random, and the like. Flink's own output for these is non-deterministic
(it depends on wall-clock and execution timing), so byte-for-byte parity is not even well-defined. For these we use
our own reasonable implementation rather than trying to mirror Flink exactly, and we do NOT gate or refuse a query
just because it observes such a value — admit it like any other expression. We still replicate Flink wherever the
result IS deterministic: an operator that merely *orders by* processing time (proctime dedup / OVER) must produce
the same output as Flink, because that depends only on arrival order, not the clock value.

This is an open-source project built for the community, not just for the maintainer. Design and tooling decisions
must serve any developer who clones the repo: prefer self-contained, portable setups (e.g. a build that needs no
machine-specific install — see the bundled-static native Kafka linking) over anything that assumes the maintainer's
environment. Never bake in personal absolute paths, hostnames, credentials, or internal-infrastructure assumptions;
keep those out of the codebase and tests. When a choice trades convenience-for-me against works-for-everyone, choose
works-for-everyone.

For each commit, I want small, targeted diffs with a clear purpose. Commit messages should be used as the sole source
of truth for developer-facing documentation. They should be more architectural in nature - do not name specific
classes that the average developer does not know off the top of their head, but instead concisely explain the "why"
of the change and the reasons for our architecture.

When reviewing code, make sure that it follows the existing principles of codebases which we will take influence from:
- Flink (see ~/data/flink for code)
- DataFusion (see ~/data/datafusion for code)
- DataFusion comet (see ~/data/datafusion-comet for code)
- Arroyo (see ~/data/arroyo for code)

Reference-first rule (do this BEFORE designing, not after):
- When adding an operator that already exists in Arroyo, you MUST first consult Arroyo's
  implementation of it (~/data/arroyo) and mirror its structure, deviating only with a stated reason
  (recorded in `divergences/`). We are ripping operators out of Arroyo, not reinventing them.
- When writing code that touches JNI, native memory management, or the Java↔Rust handover (the
  Arrow C Data Interface bridge, allocator ownership, off-heap accounting), you MUST consult
  DataFusion Comet (~/data/datafusion-comet) for the established pattern before writing ours.

Because of AI code reviews, we anticipate that there will be an influx of code. I want humans to agree on the
architecture of a solution, and then allow AI to ensure that the written code is clean and readable by any human.
Comments should not be necessary unless they explain something non-obvious to the reader. Follow typical clean code
principles like DRY and KISS. All changes should be tested, and you should look for uncovered significant edge cases
in tests. When adding new functionality to accelerate streaming, we should be able to benchmark it vs. before and
add those improvements to our commit message. If our benchmarks don't improve, we should seriously reconsider whether
the feature is worth it, or if it is the precursor to more optimizations. We also need to confirm compatibility with
existing Flink results.

The `readme.md` is a **lean landing page**, not the full spec. Keep it to: what we accelerate (a short prose
overview, not a per-operator chart), where we take inspiration from, the headline Nexmark benchmark table, how to
run and configure, related work, and the license. It must NOT enumerate every accelerated operator with its terms,
and it must NOT carry the full benchmark method / every result table. Those live in the two docs it points to:
`docs/coverage-and-fallbacks.md` (the source of truth for coverage — what does and doesn't run natively, and every
fallback cause) and `docs/benchmarks.md` (benchmark method, the Criterion micro-benchmarks, and the full
end-to-end/Nexmark tables). When an operator or benchmark changes, update those docs; touch the readme only if the
high-level picture or the headline numbers change.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [datafusion-contrib/StreamFusion](https://github.com/datafusion-contrib/StreamFusion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->
