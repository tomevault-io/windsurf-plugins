---
trigger: always_on
description: `vortex_torch` is a JIT-compiled sparse-attention framework that plugs
---

# vortex_torch — project primer for Claude Code

`vortex_torch` is a JIT-compiled sparse-attention framework that plugs
into sglang's decode loop. Users / AI agents write a **sparse-attention
submission** — two files in `submissions/` — and the framework compiles
them into Triton kernels at runtime.

## Objective

> **Strike the best tradeoff between accuracy (`mean@16`) and
> decoding throughput on AIME24.**

There is **no fixed quality floor** and no single number to maximise.
Both `mean@16` and `throughput` are objectives — the goal is to push
the **Pareto frontier** outward. A variant that buys throughput at
a small `mean@16` cost is useful; a variant that buys accuracy at a
small throughput cost is useful too. Map the frontier across a
batch by varying *along* the tradeoff: accuracy-leaning knobs
(looser `vortex_topk_val`, fewer `vortex_layers_skip`, bf16 KV) on
some variants, throughput-leaning knobs (tighter `topk`, more layer
skips, fp8 `kv_cache_dtype`, `approxTopK(tolerate_ratio=…)`,
`mem_fraction_static → 0.9`) on others. Pick winners by where
they sit on the `(throughput, mean@16)` plane against the running
best in `memory.md §5`, not by clearing a fixed bar.

## Inventing beyond the literature

The `papers/` folder and [papers/guide.md](papers/guide.md) cover
what's already published — sinks, heavy hitters, channel sparsity,
low-rank K, LSH sampling, dual-band centroids. Treat them as
**seeds, not a menu.** A winning flow does not need a citation.

**Novelty bar.** Algorithmic innovation is the primary objective.
Every batch must reserve **at least one slot** (aim for two) for a
*genuinely novel* variant — an idea from §16.2 (untried knobs),
§16.3 (inversions), §16.4 (first-principles), or the framework's own
op set. Paper replicas and §16.1 combinations are catalog-adjacent
and do not fill the novelty slot.

**The remaining 2–3 slots should use `papers/guide.md` §16.5
techniques** — catalog-adjacent parameter sweeps (`vortex_topk_val`,
`approxTopK`, layer-skip patterns, fp8/bf16 KV, etc.) that are
explicitly encouraged for non-novelty slots. They map the Pareto
curve around the novel variant and give the measured context needed
to judge whether the novel idea is actually buying something.

## Where the instructions live

All authoritative content lives under [AI/](AI/). Read in order:

1. [AI/AGENTS.md](AI/AGENTS.md) — the full submission contract, rules,
   budget / BOS / layer-skip semantics, benchmark protocol.
2. [AI/tutorials/overview.md](AI/tutorials/overview.md) — 5-minute map.
3. [AI/tutorials/program_create_cache.md](AI/tutorials/program_create_cache.md)
4. [AI/tutorials/program_forward_cache.md](AI/tutorials/program_forward_cache.md)
5. [AI/tutorials/program_forward_indexer.md](AI/tutorials/program_forward_indexer.md)
6. [AI/tutorials/cache_op.md](AI/tutorials/cache_op.md) — indexer-side
   op math reference.
7. [AI/tutorials/indexer_op.md](AI/tutorials/indexer_op.md) — cache-side
   op math reference.
8. [papers/guide.md](papers/guide.md) — synthesis of the ten
   sparse-attention papers in `papers/`. §14 = catalog of
   known-good submission ideas; **§16 = prompts for inventing
   flows that no paper here explores.**

Framework-internal deep dives live in
[AI/developer_guides/](AI/developer_guides/) — needed only if you are
modifying the compiler itself, not when writing a submission.

## Hard constraints

- **No native torch ops** inside `create_cache` / `forward_cache` /
  `forward_indexer`. Every tensor goes through
  `vortex_torch.indexer.*` / `vortex_torch.cache.*` ops. `.view`,
  `.sum(dim=...)`, elementwise torch, etc. will not compile.
- **Each op instance is one call site.** `self.mul_a = Multiply()`
  and `self.mul_b = Multiply()` — do not share.
- **Do not declare `"k"` or `"v"`** in `create_cache`; they are
  auto-provided.
- **`forward_indexer` must end in `topK(score, o, ctx=ctx)` or
  `approxTopK(tolerate_ratio=...)(score, o, ctx=ctx)`** — the
  score must be RAGGED `[S, 1, 1]`. `approxTopK` is a faster
  adaptive 8-bit radix variant; `tolerate_ratio ∈ [0.0, 1.0]`
  where `0.0` = exact, higher = cheaper but looser.
  *Trtllm-only alternative:* with
  `"vortex_attention_backend": "trtllm"` you can instead end in
  `Union()((bt_a, sl_a), (bt_b, sl_b), o, ctx=ctx)` fed by two
  `TopK(k=...)(score, ctx=ctx) → (block_tables, seqlens)` calls.
  `TopK` / `Union` assert at profile time if used under flashinfer
  (see `AI/tutorials/indexer_op.md §9b`).
- **Cache-side reductions support `dim ∈ {1, 2}` only.** Cross-block
  reductions (`dim=0`) belong on the indexer side.
- **If a field is read+written across steps via `Load`/`Save`, zero
  it with `CFill(0.0)` in `forward_cache`.**
- **If `forward_indexer` uses `Save(...)`, the engine JSON MUST set
  `"disable_radix_cache": true`** (default `false`). sglang's
  prefix-radix cache otherwise shares per-request Save'd state
  across requests with matching prompt prefixes, corrupting
  Save/Load values. `check_engine_config` rejects the violation.

## Environment — activate the `vortex_v1` conda env first

Every python invocation in this project (`check_engine_config`,
`run_submission_aime24.py`, the pre-flight loops in the slash
commands, etc.) expects the **`vortex_v1`** conda environment.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Infini-AI-Lab/vortex_torch](https://github.com/Infini-AI-Lab/vortex_torch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
