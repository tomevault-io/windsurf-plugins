---
trigger: always_on
description: This is the canonical operating guide for coding agents working in this
---

# GABRIEL agent guide

This is the canonical operating guide for coding agents working in this
checkout or writing code that uses GABRIEL. Keep it concise and update it when
public behavior changes. The README explains the package; the tutorial notebook
contains runnable examples; `src/gabriel/api.py` is the source of truth for
public signatures.

The recommendations below are practical heuristics, not universal rules. They
reflect recurring successful workflows and package behavior at the date shown;
they may be stale, unavailable in a particular organization, too expensive, or
wrong for a specific dataset. User intent, the live API/package, and evidence
from a representative pilot take precedence.

Before improvising a workflow, read `gabriel_tutorial_notebook.ipynb` carefully.
It is the most complete capability map and shows how tasks, modalities, custom
endpoints, checkpoints, visualization, and downstream analysis fit together.

## Before writing or running code

- Inspect the actual environment first. Confirm `gabriel.__file__`, the
  installed `openai-gabriel` version, and the live function signature instead
  of assuming the PyPI package, this checkout, and an old example are identical.
- Inspect unfamiliar scripts and inputs before executing them. For a paid run,
  confirm the row count, prompt count, model, output directory, and expected
  cost, then pilot a small representative sample before scaling.
- Prefer the public `gabriel.*` wrappers. Use `await` in notebooks and
  `asyncio.run(...)` in ordinary Python scripts.
- Never place API keys, private data, machine-specific paths, or generated
  research artifacts in committed examples.

## Models change: verify them live

Never invent a model slug or copy one blindly from an old notebook. Check the
[official model catalog](https://developers.openai.com/api/docs/models), the
[pricing page](https://developers.openai.com/api/docs/pricing), and the models
actually provisioned for the user's organization or gateway before overriding a
default or publishing a release. Availability, aliases, rate limits, and feature
support can vary by organization.

At this guide's last review (2026-07-13), GABRIEL used this snapshot:

| Workload | Exact model slug | Typical use |
| --- | --- | --- |
| Fast and inexpensive | `gpt-5.6-luna` | Often a sensible starting point, especially for large-scale runs; default for rate, classify, rank, extract, filter, and merge |
| Balanced | `gpt-5.6-terra` | Consider when a representative pilot materially improves over Luna and does not require Sol |
| Full capability | `gpt-5.6-sol` | Very complex or subtle work with many moving parts, niche knowledge, difficult synthesis, or important writing quality |
| Audio understanding | `gpt-audio-1.5` | Audio input through Chat Completions |

These are dated facts, not permanent aliases. Verify that a prospective model
supports the required endpoint, modality, reasoning setting, and structured
output behavior. Do not mechanically replace an audio model with a text model.

A common starting point is Luna, particularly at scale. If quality is uncertain,
run a small representative comparison across Luna, Terra, and Sol and inspect
task-specific accuracy, omissions, consistency, latency, and cost. Prefer the
smallest model that reliably meets the research standard. Escalate to Sol when
the work is genuinely complex or subtle, has many interacting constraints,
depends on niche internal knowledge, or when the quality of the reasoning or
writing is itself important. Do not choose model size mechanically from the
GABRIEL function name alone.

## Defaults and scale heuristics

- Generally keep `n_parallels=650`, including for small tests. It is a ceiling,
  not a fixed worker count, so a small job does not create 650 unnecessary
  requests. GABRIEL ramps workers, observes rate limits, retries transient
  failures, and reduces actual concurrency for sustained errors, web search,
  and media. Temporary retries and a few slow stragglers are normal; while a run
  is progressing, it is usually best to let it finish. Raising the ceiling can
  be reasonable when a large workload needs more throughput and the account or
  gateway supports it. Reducing it can be appropriate for a known deployment
  constraint or persistent failure, but is rarely needed merely because a test
  is small. These are operational suggestions, not hard limits.
- Do not pass `max_output_tokens`. GABRIEL accepts it only for compatibility,
  emits a `FutureWarning`, and ignores it.
- Keep `reset_files=False` to resume a compatible checkpoint. If the model,
  prompt, labels, attributes, batching, or meaning of a run changes, use a new
  `save_dir` or intentionally reset; never mix two specifications in one cache.
- Normally leave `n_attributes_per_run=None`. Current models can often handle
  dozens of labels or fields together—including 30, 40, or 50—while retaining
  useful cross-attribute context. The parameter remains available if an actual
  pilot reveals a context, schema-adherence, quality, or provider constraint;
  do not split attributes preemptively.
- Reuse GABRIEL's retries and checkpoints. Do not wrap it in ad hoc thread pools
  or rerun an entire corpus when only a small failed subset needs repair.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openai/GABRIEL](https://github.com/openai/GABRIEL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
