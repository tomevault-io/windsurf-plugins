---
trigger: always_on
description: Blink is a one-pass typed-decision model with an embeddable C runtime. The C
---

# AGENTS.md

Blink is a one-pass typed-decision model with an embeddable C runtime. The C
library in `src/` and `include/` is the product. The Python in `python/`,
`eval/` and `scripts/` exists to produce the weights it runs and to measure it.

## Verify a change

```sh
make clean && make && make test        # C: no Python needed
PYTHONPATH=python .venv/bin/python -m pytest tests/python -q
make bench
```

`scripts/run_all.sh` is the release test. Run it before changing any number in
`docs/RESULTS.md`; `scripts/run_all.sh --smoke` runs every stage in minutes and
is what to run after a change to the pipeline itself.

## Invariants that must not regress

- **The scoring path allocates nothing.** `tests/c/check_no_malloc.sh` asserts
  that `blink_runtime.o` and `blink_kernels.o` reference no allocator symbol.
  If you need a helper that allocates, put it in `src/blink_alloc.c`.
- **Four implementations of one forward pass must agree.** `src/blink_kernels.c`
  and `src/blink_runtime.c` (C, fp32), `python/blink_train/reference.py`
  (NumPy, float64, the normative definition), `python/blink_train/model.py`
  (PyTorch, batched and padded) and `tools/blink_synth.c` (the C container
  writer). Change one and you change all of them; `tests/python/test_parity.py`
  is what catches a partial change.
- **Reusing an encoded state is bit-identical to re-encoding it.** This is the
  point of encoding the state and the question as separate sequences. It is
  checked with `memcmp`, not a tolerance.
- **The build pins `-ffp-contract=off`.** Removing it invalidates the parity
  tolerance.
- **Every geometry field is validated on open.** A malformed container must be
  refused with a specific status, never trusted. Adding a header field means
  adding a rejection test for it in `tests/c/test_container.c`.
- **The container header layout is append-only.** Fields live at fixed byte
  offsets read in three places: `parse_header` in `src/blink_model.c`,
  `ContainerWriter._header` in `python/blink_train/container.py`, and `main` in
  `tools/blink_synth.c`. New fields go in the reserved space and bump nothing.
- **Changing what a tensor means bumps the format version.** A new dtype, a new
  shape or a new computation over an existing tensor must bump
  `BLINK_FORMAT_VERSION` in `src/blink_internal.h` and `FORMAT_VERSION` in
  `python/blink_train/container.py` together, and `tests/c/test_container.c`
  must reject the previous version. Otherwise an old container loads and
  silently computes something else. It is 3 (the cosine option head).
- **Every option logit is bounded.** The head is a sum of per-head cosines
  times one scale, so `|logit| ≤ heads × |logit_scale|` whatever the
  projections learn. Do not reintroduce a raw dot product anywhere in the
  head; `test_parity.py` checks the bound with thousand-fold inflated weights.
- **A `temperature_pinned` warning is a failed run.** It means the temperature
  fit hit its bound, which is what a diverging head looks like from outside.
  Read the training log before quoting any number from such a run.

## What has been tried

- **A flat stack of feed-forward blocks at full byte resolution** made the
  `small` preset take roughly a second per state encode. Pooling after a cheap
  stem is what makes byte-level affordable; keep the expensive part behind the
  pool.
- **One seed proves nothing here.** The FiLM ablation was run single-seed three
  times and gave three different answers: FiLM rescues `judgment`, FiLM rescues
  `comparison`, FiLM hurts. A three-seed run settled it as *undecided* on every
  slice, because the within-arm spread is larger than the between-arm gap. Run
  several runs (`scripts/experiments/` has the queues) and
  `scripts/seed_report.py` before claiming any
  architectural change helps.
- **`judgment` is bimodal and that is the source of the variance.** A run either
  learns to match the verb in the question against the verb in the state or it
  does not; the failing mode collapses *supports* and *contradicts* and scores
  exactly 2/3. Nothing else in the corpus behaves this way.
- **Every residual sub-layer starts as an identity.** The output projection of
  the mixer and of the cross layer, and both FiLM matrices, are zero-
  initialised. A sub-layer added with a random output projection perturbs a
  working residual path from step one; that cost this model tens of epochs in a
  worse optimum than it started in, on three separate occasions, before the
  pattern was recognised. If you add a sub-layer, zero its output.
- **FiLM weights are zero-initialised and stay fp32.** Random initialisation
  perturbs a working residual path from step one. Quantizing them is worse:
  they scale the option vectors, so the error is multiplicative, and enabling
  quantization-aware training drove validation NLL from 0.87 to 1.51. Both
  facts are load-bearing; do not "tidy" either away.
- **Entity pools are split three ways, not two.** The calibration temperature is
  fitted on validation, so validation must also be out-of-vocabulary. A two-way
  split produced 0.69 accuracy with an ECE of 0.13 purely from that mismatch.
- **Higher learning rates do not help the hard families.** At 6e-3 and 1e-2 the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sqliteai/blink](https://github.com/sqliteai/blink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
