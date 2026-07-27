---
trigger: always_on
description: **Purpose.** This file tells Claude (or any LLM assistant) exactly how to help on this repo without wasting maintainer time. It encodes project norms, API/persistence guarantees, review rubrics, performance boundaries, and Scala engineering standards.
---

# CLAUDE.md — Maintainer Copilot for *generalized-kmeans-clustering*

**Purpose.** This file tells Claude (or any LLM assistant) exactly how to help on this repo without wasting maintainer time. It encodes project norms, API/persistence guarantees, review rubrics, performance boundaries, and Scala engineering standards.

> TL;DR
> **Primary surface:** Spark **DataFrame/ML API** (`GeneralizedKMeans`, etc.) — **RDD API removed in v0.7.0**.
> **Versions:** Scala **2.13** (primary) / 2.12, Spark **4.0.x / 3.5.x / 3.4.x**
>   - **Spark 4.0.x**: Scala 2.13 only (2.12 dropped in Spark 4.0)
>   - **Spark 3.x**: Both Scala 2.13 and 2.12 supported
> **Math:** Bregman family — divergences include `squaredEuclidean`, `kl`, `itakuraSaito`, `l1`, `generalizedI`, `logistic`, `spherical`/`cosine`.
> **Variants:** Bisecting, X-Means, Soft/Fuzzy, Streaming, K-Medians, K-Medoids.
> **Determinism + persistence** are non-negotiable.
> **Architecture:** Modular package structure with `kernels/` and `strategies/impl/` subpackages.

---

## 0) Operating Principles (do these every time)

1. **Use the DataFrame/ML API.** Code and examples use Estimator/Model patterns and Params from this codebase. (RDD API was removed in v0.7.0.)
2. **No silent API breaks.** If you touch params, model JSON, or persistence schemas, include migration/round-trip tests.
3. **Mathematical fidelity first.** Correct Bregman formulations beat micro-perf. Perf changes must not alter semantics.
4. **Determinism matters.** Same seed ⇒ identical results. Avoid nondeterministic ops in core loops.
5. **Tight PRs.** Small, test-backed, CI-friendly. No speculative abstractions.
6. **Maintain the roadmap.** Keep `ROADMAP.md` forward-looking; move completed work into `CHANGELOG.md` and leave the roadmap focused on upcoming items and priorities.
7. **Use shared model helpers.** Clustering models should mix in `HasTrainingSummary` and `CentroidModelHelpers` for consistent summaries/metadata; avoid reintroducing ad-hoc summary fields.

---

## 0.1) Roadmap Maintenance

**IMPORTANT:** The file `ROADMAP.md` contains the project's technical roadmap, including:
- Bug fixes (completed and pending)
- Architecture improvements
- Algorithm additions
- Performance improvements
- Documentation needs

**Claude must:**
1. **Inspect `ROADMAP.md`** at the start of significant work to understand current priorities and context.
2. **Update `ROADMAP.md`** when:
   - Discovering a new bug or opportunity → add with priority
   - Changing priorities → update ordering/sections
   - Making architectural decisions → add to Decision Log
3. **Move completed work** (features, fixes, docs, perf) into `CHANGELOG.md` with dates and drop it from `ROADMAP.md` so the roadmap stays forward-looking.
4. **Reference roadmap items** in commit messages and PR descriptions where applicable.

---

## 1) Project Snapshot (facts Claude must anchor to)

- **Surface:** `GeneralizedKMeans` (and friends) via DataFrame API; prediction via `transform`.
- **Spark:** 4.0.x / 3.5.x / 3.4.x all tested.
  - **Spark 4.0.x**: Scala 2.13 only (Scala 2.12 dropped in Spark 4.0)
  - **Spark 3.x**: Both Scala 2.13 and 2.12 supported
- **Scala:** 2.13.x primary (keep code Scala-3-friendly where feasible).
- **Java:** 17.
- **Divergences:** `squaredEuclidean | kl | itakuraSaito | l1 | generalizedI | logistic | spherical | cosine`.
- **Assignment strategies:** `auto | crossJoin (SE fast path) | broadcastUDF (general Bregman)`.
- **Input transforms:** `none | log1p | epsilonShift(shiftValue)`; ensure domain validity for KL/IS.
- **Persistence:** Models round-trip across Spark 3.4↔3.5↔4.0, Scala 2.12↔2.13.
- **Package structure:**
  - `clusterer.ml.*` — Top-level algorithm estimators & models
  - `clusterer.ml.df.*` — DataFrame API implementation
  - `clusterer.ml.df.kernels.*` — 8 Bregman kernel implementations
  - `clusterer.ml.df.strategies.*` — Assignment/update strategies
  - `clusterer.ml.df.strategies.impl.*` — 5 assignment strategy implementations

---

## 2) What Good Help Looks Like

Claude should deliver one of:
- **Small PR plan** with diff outline + tests.  
- **Minimal patch** that compiles locally and within the CI matrix.  
- **Review comments** (actionable, line-anchored).  
- **Doc/example edits** with runnable snippets.

Always include: **test list**, **risk matrix (API/persistence)**, **rollback note**.

---

## 3) Common Tasks & Prompts (ready to paste)

### (A) Add/adjust a DF Param
> “Propose a minimal patch to add `<paramName: type>` to `GeneralizedKMeans` with default `<value>`, update model persistence JSON, validate in `ExamplesSuite`, and write a property test. Include compatibility notes and a unified diff.”

### (B) SE fast path perf (crossJoin)
> “Audit the squared-Euclidean fast path for shuffles/sorts. Propose a plan using `groupBy(rowId).min(distance)` + `join` for argmin where beneficial. Provide an explain plan and a micro-benchmark kept under 30s in CI.”

### (C) Streaming snapshots

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [derrickburns/generalized-kmeans-clustering](https://github.com/derrickburns/generalized-kmeans-clustering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
