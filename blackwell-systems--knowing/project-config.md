---
trigger: always_on
description: Self-adapting code intelligence engine. Single Go binary, zero runtime deps.
---

# knowing

Self-adapting code intelligence engine. Single Go binary, zero runtime deps.
Gets smarter with scale, not dumber: observes its own graph density and adjusts
retrieval strategy automatically. 38 edge types, 23 extractors, 28 MCP tools.

## Build & Test

```bash
GOWORK=off go build ./...           # build (GOWORK=off required: go.work refs missing module)
GOWORK=off go test ./internal/...   # unit tests
GOWORK=off go test ./cmd/...        # CLI tests
GOWORK=off go test ./bench/...      # benchmark harnesses (some need pre-indexed repos)
```

## Benchmark (P@10 evaluation)

```bash
# Full corpus, sequential (official numbers, ~20 min with pre-embedded vectors)
BENCH_EMBEDDINGS=1 BENCH_ADAPTERS=knowing GOWORK=off go test ./bench/cross-system/ -run TestCrossSystem -v -timeout 0

# Full corpus, parallel (iteration mode, ~5 min, P@10 ~0.022 lower due to ONNX CPU contention)
BENCH_PARALLEL=1 BENCH_EMBEDDINGS=1 BENCH_ADAPTERS=knowing GOWORK=off go test ./bench/cross-system/ -run TestCrossSystem -v -timeout 0

# Single repo (fast iteration, no embeddings)
BENCH_REPOS=django BENCH_ADAPTERS=knowing GOWORK=off go test ./bench/cross-system/ -run TestCrossSystem -v -timeout 10m

# With embeddings on single repo (~2 min with pre-embedded vectors)
BENCH_EMBEDDINGS=1 BENCH_REPOS=django BENCH_ADAPTERS=knowing GOWORK=off go test ./bench/cross-system/ -run TestCrossSystem -v -timeout 30m

# Pre-embed all nodes (one-time, ~2 hours for full corpus, skips phantoms)
knowing enrich embeddings -db <repo>/.knowing/graph.db

# Diagnostic env vars (compose freely, no reindex needed):
BENCH_EXCLUDE_EDGES=similar_to,type_hint_of   # exclude edge types from RWR walk
BENCH_BFS_DEPTH=2                             # limit walk depth (default 4)
BENCH_PREFER_TYPE_SEEDS=1                     # force type-seed preference
BENCH_HUB_DAMPEN=50                           # penalize nodes with in-degree >50
BENCH_RERANK_WEIGHT=0.5                       # blend original + embedding scores
BENCH_COHERENCE_BONUS=0.3                     # file-based packing coherence
BENCH_MAX_SEEDS=25                            # override max seed count
BENCH_ADAPTIVE_SEEDS=1                        # enable adaptive seed count
BENCH_GAP_THRESHOLD=5                         # gap-fill activation threshold (default 5)
BENCH_PARALLEL=1                              # parallel repo execution (fast, ~0.022 P@10 lower)
```

## Testing Methodology

Django is the acid test repo for retrieval experiments:
- 33 tasks (largest single-repo fixture set)
- 42% zero-rate problem (vocabulary gaps), so improvements that move Django are structural
- Where adaptive seeds showed +14.2%, bidirectional inheritance showed -2.5%, gap injection +3.2%

**Protocol:**
1. **Django only, no embeddings (~30s):** quick signal on structural changes
   ```bash
   BENCH_REPOS=django BENCH_ADAPTERS=knowing GOWORK=off go test ./bench/cross-system/ -run TestCrossSystem -v -timeout 10m
   ```
2. **Django with embeddings (~7min):** confirms interaction with re-ranker
   ```bash
   BENCH_EMBEDDINGS=1 BENCH_REPOS=django BENCH_ADAPTERS=knowing GOWORK=off go test ./bench/cross-system/ -run TestCrossSystem -v -timeout 30m
   ```
3. **Full corpus with embeddings (~80min):** only if Django moves positively
   ```bash
   BENCH_EMBEDDINGS=1 BENCH_ADAPTERS=knowing GOWORK=off go test ./bench/cross-system/ -run TestCrossSystem -v -timeout 0
   ```

If Django is neutral or negative, don't run the full corpus. If Django is positive,
the full corpus confirms whether it generalizes or gets absorbed by run variance.

**Important:** Not all experiments affect Django. Check graph density first:
```bash
sqlite3 <repo>/.knowing/graph.db "SELECT COUNT(*) FROM edges; SELECT COUNT(*) FROM nodes;"
```
Density = edges/nodes. Current densities: cargo 13.5, kafka 12.5, terraform 9.5,
ocelot 8.3, spark-java 7.7, kubernetes 6.2, flask 5.9, vscode 4.7, django 2.8.
If the experiment only affects dense graphs (like adaptive alpha), test on dense repos
(flask, cargo, kafka) instead of Django.

**Output capture:** Always capture full output to a file (`2>&1 | tee /tmp/file.log`
or `> /tmp/file.log 2>&1`). Never pipe through `tail` as it loses early output.

## When Benchmark Numbers Change

After any P@10 improvement, these files ALL need updating with the new aggregate,
per-repo breakdown, and competitive ratios. This is a standard procedure:

1. **bench/cross-system/FINDINGS.md** — executive summary, per-repo table, competitive advantages
2. **bench/CONTEXT-PACKING-STUDY.md** — Dimension 1, competitive summary, run history
3. **bench/README.md** — cross-system row in summary table
4. **docs/guide/introduction.md** — operational characteristics, measured performance table
5. **docs/architecture/retrieval-pipeline.md** — eval baseline line
6. **docs/architecture/system-overview.md** — benchmark section
7. **docs/architecture/design-principles.md** — benchmark results
8. **docs/architecture/context-engine.md** — current performance
9. **docs/architecture/embedding-reranker.md** — impact numbers
10. **docs/roadmap.md** — retrieval pipeline section
11. **docs/research/cross-system-benchmark.md** — key results table

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blackwell-systems/knowing](https://github.com/blackwell-systems/knowing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
