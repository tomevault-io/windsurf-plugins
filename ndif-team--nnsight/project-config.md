---
trigger: always_on
description: This file routes you to the right documentation under `docs/` for whatever the user is asking about. The actual content lives in `docs/`. **Read the relevant doc page before writing code** — these docs are tight, recipe-style, and frequently updated to match the current branch.
---

# nnsight — Agent Guide

This file routes you to the right documentation under `docs/` for whatever the user is asking about. The actual content lives in `docs/`. **Read the relevant doc page before writing code** — these docs are tight, recipe-style, and frequently updated to match the current branch.

If you're new to nnsight, read [docs/concepts/index.md](docs/concepts/index.md) once. Otherwise jump directly to the doc that matches the user's task.

---

## How to use this file

1. Find the user's intent in **"By task"** below and follow the link.
2. If the user's request maps to a model class (LanguageModel, VLLM, etc.), check **"By model class"**.
3. If something is broken, check **"Errors"** or **"Gotchas"**.
4. The **"Inline gotcha cheat-sheet"** at the bottom catches the most common agent mistakes — internalize them before writing any nnsight code.

---

## By task

### "I want to read activations / modify them on a single forward pass"
- [docs/usage/trace.md](docs/usage/trace.md) — `model.trace(input)`
- [docs/usage/access-and-modify.md](docs/usage/access-and-modify.md) — `.output`, `.input`, `.inputs`, in-place vs replacement
- [docs/usage/save.md](docs/usage/save.md) — keep values past the trace exit

### "I want multi-token / autoregressive generation"
- [docs/usage/generate.md](docs/usage/generate.md) — `model.generate(input, max_new_tokens=N)`
- [docs/usage/iter-all-next.md](docs/usage/iter-all-next.md) — `tracer.iter[...]`, `tracer.all()`, `tracer.next()` / `module.next()`

### "I want to run multiple prompts at once"
- [docs/usage/invoke-and-batching.md](docs/usage/invoke-and-batching.md) — `tracer.invoke(...)`, batched lists, empty invokes
- [docs/usage/barrier.md](docs/usage/barrier.md) — `tracer.barrier(n)` for cross-invoke value sharing
- [docs/patterns/multi-prompt-comparison.md](docs/patterns/multi-prompt-comparison.md)

### "I want to look inside a module's forward (intermediate operations)"
- [docs/usage/source.md](docs/usage/source.md) — `model.<path>.source.<op_name>.output / .input`
- [docs/concepts/source-tracing.md](docs/concepts/source-tracing.md) — how `.source` rewrites the AST

### "I want to cache activations from many modules"
- [docs/usage/cache.md](docs/usage/cache.md) — `tracer.cache(modules=..., include_inputs=...)`

### "I need gradients / backward pass"
- [docs/usage/backward-and-grad.md](docs/usage/backward-and-grad.md) — `with tensor.backward():` is its own session
- [docs/patterns/gradient-based-attribution.md](docs/patterns/gradient-based-attribution.md)
- [docs/patterns/attribution-patching.md](docs/patterns/attribution-patching.md)

### "I want to run remotely on NDIF"
- [docs/remote/ndif-overview.md](docs/remote/ndif-overview.md) — what NDIF is, job lifecycle
- [docs/remote/api-key-and-config.md](docs/remote/api-key-and-config.md) — set up your API key
- [docs/remote/remote-trace.md](docs/remote/remote-trace.md) — `model.trace(..., remote=True)`
- [docs/remote/remote-session.md](docs/remote/remote-session.md) — bundle multiple traces into one job
- [docs/remote/non-blocking-jobs.md](docs/remote/non-blocking-jobs.md) — submit and poll
- [docs/remote/register-local-modules.md](docs/remote/register-local-modules.md) — ship local code to NDIF

### "I want to verify shapes / inspect dimensions without running the model"
- [docs/usage/scan.md](docs/usage/scan.md) — `model.scan(...)`
- [docs/gotchas/save.md](docs/gotchas/save.md) (`.save()` still required inside scan)

### "I want to make persistent edits to a model"
- [docs/usage/edit.md](docs/usage/edit.md) — `model.edit()` / `model.edit(inplace=True)`

### "I want to run a research pattern (logit lens / patching / steering / SAE...)"
- [docs/patterns/index.md](docs/patterns/index.md) — full cookbook
- Most-asked-for: [logit-lens](docs/patterns/logit-lens.md), [activation-patching](docs/patterns/activation-patching.md), [ablation](docs/patterns/ablation.md), [steering](docs/patterns/steering.md), [attention-patterns](docs/patterns/attention-patterns.md), [sae-and-auxiliary-modules](docs/patterns/sae-and-auxiliary-modules.md), [per-head-attention](docs/patterns/per-head-attention.md)

### "I'm extending nnsight (custom Envoy / new runtime / new value type)"
- [docs/usage/extending.md](docs/usage/extending.md) — `envoys=` kwarg + custom `eproperty`
- [docs/concepts/envoy-and-eproperty.md](docs/concepts/envoy-and-eproperty.md) — mental model
- [docs/developing/eproperty-deep-dive.md](docs/developing/eproperty-deep-dive.md) — full extension API

### "Something is broken / I got an error"
- [docs/errors/index.md](docs/errors/index.md) — exception → cause → fix table
- [docs/gotchas/index.md](docs/gotchas/index.md) — most common ways things go wrong
- [docs/errors/debug-mode.md](docs/errors/debug-mode.md) — turn on full tracebacks

---

## By model class

| Need | Use | Doc |
|---|---|---|
| Any `torch.nn.Module` | `NNsight(module)` | [docs/models/nnsight-base.md](docs/models/nnsight-base.md) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ndif-team/nnsight](https://github.com/ndif-team/nnsight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
