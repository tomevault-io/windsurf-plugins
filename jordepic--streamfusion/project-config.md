---
trigger: always_on
description: This project attempts to run Apache Flink using Apache DataFusion + rust operators via JNI to accelerate stream
---

This project attempts to run Apache Flink using Apache DataFusion + rust operators via JNI to accelerate stream
processing throughput. It is a very similar project to DataFusion comet in spirit, but with stream processing.

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
existing Flink results. The readme.md should include our benchmarks for each operator we implement as well as a
comparison to the flink operator.

Builds: tests run against a debug native build for a fast iteration loop (`mvn test`), but ALL benchmarking
must use the release build via the `bench` Maven profile (`mvn test -Pbench ...`) — debug Rust is roughly an
order of magnitude slower and gives misleading numbers. Never report a benchmark from a debug build.

At a high level:
We are ripping code out of Arroyo, which itself already uses DataFusion
We are overriding the planning layer of Flink to use our Arroyo operators
This allows us to handle incoming records is arrow batches as opposed to using the flink row model

**Native operators are columnar.** Every native operator we add should consume and produce Arrow
column batches (`ArrowBatch`) — i.e. implement `ColumnarInput`/`ColumnarOutput` — not Flink `RowData`.
The *only* exceptions are the transpose operators, which exist precisely to bridge the two models at
the native↔host boundary. The row↔Arrow conversion is paid once at the edges (the transition pass
inserts a transpose where a columnar operator meets a rowwise host operator), never baked into an

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jordepic/StreamFusion](https://github.com/jordepic/StreamFusion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
