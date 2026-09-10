---
trigger: always_on
description: Every number in the README comes from the scripts in this folder run
---

# Working in this folder — the usage model, the footprint, the recipe

Every number in the README comes from the scripts in this folder run
against **unmodified host checkouts**. Read this before touching or
reproducing anything: it states what each tier of the library actually
requires from a user, what each script touches, and exactly how large
the footprint is.

## The usage model, tier by tier

**Automatic** — install once, then two calls, nothing else:

```bash
pip install flash-rt kernels   # hub kernel packages fetch per host at first bind
```

```python
plan = structures.auto_swaps(model, run_once)   # run_once: the host's hot path, once
handle = swap.attach(model, plan.swaps, observe=plan.observed,
                     revert=plan.revert)
```

Zero host-source changes, zero files edited, nothing forked. The user
supplies the loaded model and one callable that runs its hot path on a
representative observation (that single pass is the calibration).
Everything is in-process and `handle.detach()` restores the host
bit-for-bit. This buys the eager/compiled band with per-seam guards
and a ledger.

**Captured (full speed)** — one more mechanism call, not a harness:

```python
stage = structures.capture(torch.compile(hot), model=model)
stage.replay()
```

When `capture` is handed the model, the registered host-family
lowering adapter recognizes it, records one real request, pins that
family's shape glue (both transformers vision-contract generations,
plus wrapper glue via capability probe), and writes the family and its
pins into the stage certification. `stage.restore_host()` takes every
pin back off. The automatic and explicit assemblies reach their
fastest numbers through this same door; a host no family recognizes is
captured as-is, and a family that cannot pin safely refuses with a
reason instead of leaving the host half-pinned.

**Weight residency (a lifecycle, not a mode)** — attach → validate →
`handle.consume()` → optionally `handle.finalize()`. There is no
resident tier: after your parity gate passes, `consume()` moves every
replaced original's truth off the device — to the checkpoint file when
provenance verifies (a sampled-block match against the live tensor),
to pinned host RAM otherwise — and frees its device storage. The
receipt names bytes freed and the tier split. Fallback and `detach`
survive as restore-from-store: a seam called outside its contract
restores its host once (the ledger notes `restored_for_fallback`), and
`detach` reloads before it puts the host back, still bit-exact. Seats
that actively serve through their retained host (a cadence bank
refreshing through the host projection) declare `_frt_host_serving`
and are kept whole — the receipt counts them. `handle.finalize()`
then drops the restore tickets: fallback flips to refusal, `detach`
is forbidden, irreversible and recorded as such. Consumption comes
after validation because until then the attached model still owes the
host schema (state_dict, A/B reference arms, and any capture that
aliases host weights dies the moment its pointers are freed).
Seat scratch (sibling stashes, producer workspaces, the packed-output
and quantize scratch, wire buffers) is pooled by shape: sequential
layers share one buffer, so the memory bill is one layer's worth, not
layers x tokens (`structures.workspace.report()` is the receipt's
memory column). Binding itself is budgeted: under 512 MiB free VRAM a
seat refuses with `insufficient_vram(...)` instead of eating the
remainder. The pool's safety argument is single-stream sequential
execution — leasing from a non-default CUDA stream refuses loudly.

**Receipts (one document)** — `handle.manifest()` answers "why does
this box run this form" in one serializable dict: device fingerprint,
every seam with kind/calls/fallbacks/notes, the band decisions this
device consumed, the workspace ledger, and the weight-residency
receipt. Captured windows are strict at the door: `stage.write(name,
value)` requires the exact shape/dtype/device the graph was captured
with — a broadcastable-but-wrong write refuses instead of replaying
over coerced data.

**Transaction boundary** — `attach` is the commit point. A bind that
dies midway rolls the whole plan back (routes disabled, host
mutations reverted, streamed weights restored); a caller that decides
not to commit calls `plan.abort()`. There is no half-routed state to
clean up by hand.

**Explicit** — the only tier with real orchestration in user code:
seat tables, the author's own calibration hooks, direct binder calls —
`build()`, 215 lines. It exists for control the automatic
qualification will not exercise: claiming seats discovery refuses,
writing a producer negotiation out as code, choosing a scheme per
seat.

The explicit DiT band form (FP8 chains versus the FP4 wire) is an
author pin backed by captured-form receipts, selected by
`FRT_DIT_BAND` (`fp4` default — the Thor-measured winner; `fp8` — the
5090-measured winner) until the band-level adjudicator with its
decision cache lands. Seat-level micro-timing was refuted in both
directions by production-form measurement; the design doc in the
records repo carries the receipts.

## The footprint

| where | what changes | size |
|---|---|---|
| official Isaac-GR00T source | **nothing** | 0 lines |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LiangSu8899/FlashRT](https://github.com/LiangSu8899/FlashRT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
