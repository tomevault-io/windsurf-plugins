---
trigger: always_on
description: Instructions for AI agents maintaining this repository.
---

# AGENTS.md

Instructions for AI agents maintaining this repository.

Read this before touching anything. The repository is deliberately structured so
that many agents can work on it at once without conflicting, and most of that
structure only works if you follow the file-per-record rule below.

**The one rule that matters most:** one record, one file. A model, an engine, a
use case and an issue tracker each own exactly one file. If your change touches
one model, it should touch one file under `data/models/`. If you find yourself
editing a shared file to add a model, you have taken a wrong turn.

**Before you open a PR:**

```sh
python3 tracker/validate.py     # must print "0 errors"
python3 tracker/build.py        # must succeed
python3 tools/check_output.py   # checks the rendered page
```

That is exactly what CI runs, plus an import smoke test. Warnings from
`validate.py` do not block anything — they flag a record that is merely thin.

---

## 1. Why the structure is what it is

The data used to live in two files: a 1,500-line `engines.py` holding every
model's per-engine status, and a 1,900-line `render_status.py` holding both the
model list and the entire HTML template. Two agents adding two different models
conflicted every time, and an agent fixing CSS conflicted with an agent adding a
benchmark score.

Now:

```
data/
  models/<id>.py        one model: identity, scores, per-engine status, quant ladder, KV
  engines/<id>.py       one engine: what it is, its API, its website, its
                        cross-cutting issues
  use_cases/<id>.py     one "What for?" category and its curated ranking
  issues/<owner>__<repo>.py   tracked issues for one upstream repository
  pr_keys.py            which issue keys are pull requests (small, shared)

tracker/
  registry.py           loads data/ and assembles it. The only file that knows the schema.
  validate.py           enforces the schema. CI runs this.
  render_status.py      the HTML template and the page's prose. No per-model data.
  bands.py              fidelity thresholds and per-model quant caveats
  build.py              renders docs/index.html
  measure.py            re-measures one model's quant ladder from Hugging Face
  probe.py              re-polls tracked issue states from GitHub
  watch-state.txt       last polled state of every tracked issue
  watch.sh              local twice-daily watch loop

tools/
  check_output.py       sanity-checks the rendered page

.github/workflows/
  validate.yml          the gate: validate, import-smoke, build, check output
  deploy.yml            builds and publishes Pages on push to main
  refresh.yml           re-polls issue states twice daily and commits changes
```

Conflict surface by task:

| Task | Files you touch |
|---|---|
| Add a model | `data/models/<new>.py`, and one line in one `data/use_cases/*.py` per category it belongs to |
| Correct a model's figure or prose | `data/models/<id>.py` only |
| Add or update a tracked issue | `data/issues/<repo>.py` only |
| Add an engine | `data/engines/<new>.py`, plus one cell in each `data/models/*.py` it can load |
| Change the UI | `tracker/render_status.py` only |
| Change the schema | `tracker/registry.py` **and** `tracker/validate.py`, in a commit of their own |

Two agents adding two models never touch the same file. Two agents adding issues
to two different upstream repos never touch the same file. That is the point.

---

## 2. Finding new models

There are two different jobs here, and they want different sources. **Discovery**
is "what shipped that we don't have yet". **Confirmation** is "can anything on
this page actually load it". Discovery sources are noisy and must never be
trusted on their own; confirmation sources are authoritative.

### Discovery: sources that announce new models

Every URL below was checked live on 2026-08-27. All are pollable without a key
unless noted. Recheck rather than assume — feeds move.

**Poll these first. They are structured, and they are where a new model shows up
before anyone writes about it.**

| Source | Query | What it is good for |
|---|---|---|
| Hugging Face trending | `https://huggingface.co/api/models?sort=trendingScore&direction=-1&limit=50` | The single highest-signal feed. A brand-new flagship trends within hours, at three-digit download counts. |
| Hugging Face, newest by format | `.../api/models?filter=mlx&sort=createdAt&direction=-1&limit=50` (also `filter=gguf`) | New *quants*, which is the thing that decides whether a model is usable here. |
| mlx-community uploads | `.../api/models?author=mlx-community&sort=createdAt&direction=-1&limit=50` | Apple-silicon-specific. If it appears here, someone has already converted it. |
| Ollama, newest | `https://ollama.com/search?o=newest` | A curated catalogue with a short list — parse `/library/<name>` hrefs. Very low noise. |
| llama.cpp arch list | `https://api.github.com/repos/ggml-org/llama.cpp/commits?path=src/llama-arch.cpp` | An architecture gaining support, from the authoritative file. Also names the arch string. |
| mlx-lm model classes | `https://api.github.com/repos/ml-explore/mlx-lm/commits?path=mlx_lm/models` | Same, for MLX. Note mlx-lm's *releases* lag badly (v0.31.3 is from April), so commits are the signal, not tags. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dreamingwell/apple-llm-performance](https://github.com/dreamingwell/apple-llm-performance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
