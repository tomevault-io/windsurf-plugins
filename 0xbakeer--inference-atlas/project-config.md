---
trigger: always_on
description: validates the files and builds the site; it never starts an engine and never runs a workload.
---

# AGENTS.md — the contribution contract

This file is for coding agents (and the people supervising them). If you were handed an
Inference Atlas packet, read this once, in full, before you touch anything. It is short on
purpose. `docs/SPEC.md` is the long version and wins wherever the two disagree.

## What this repository is

Inference Atlas is a map of the _configuration space_ of LLM inference: model × quantization
× hardware × engine version × flags × workload. Every square that somebody has measured shows
real numbers, attributed to the GitHub user who ran them. Every square nobody has measured is
a gap, and gaps come with a packet like the one you were given.

**The repository is the database.** There is no backend. Every measurement is one JSON file
committed to `main`, and the website is a static build of those files. That has one enormous
consequence for you:

> **Every result file has exactly one owner: the person whose GitHub login is in
> `provenance.github_login`. You may add your own. You may never modify or delete somebody
> else's.** CI enforces this on every pull request.

That rule is what makes merge conflicts structurally impossible and what makes the data
trustworthy. It is not negotiable and there is no situation in which working around it is the
right call.

**Nothing here is measured by CI.** Every number in the atlas was produced on a contributor's
own machine, by that contributor, and submitted as a pull request by that contributor. CI
validates the files and builds the site; it never starts an engine and never runs a workload.
There is no seed data either: an empty square means nobody has run it yet. So the last step
of your job is opening the pull request with the result files in it — until then the run only
exists on your disk.

## The rules

1. **Only add files you own.** Your own result files under `results/`, plus registry files
   (hardware, model, quant, engine) for things that genuinely do not exist yet. Nothing else.
2. **Never edit a number by hand.** If validation fails, fix the run or fix the metadata —
   never the measurement. A hand-patched number is data corruption with extra steps.
3. **Never silently lower the configuration.** If the requested flags do not fit — OOM,
   unsupported quantization, context too long — that _is_ the result. Record it as a failure
   with the actual error. Only then, as a _separate_ result, record what did fit, with `args`
   showing exactly what you changed and `provenance.notes` saying why.
4. **Report failures as failures.** An OOM, a crash, a 0 % success rate, an engine that will
   not start: all of these are valid, wanted contributions. A failure you quietly dropped is
   worse than no contribution at all, because it makes the map look explored when it is not.
5. **Run on an idle box.** No other GPU work, no compile in the background, no second model
   resident unless you say so. Put the conditions in `provenance.notes`: ambient temperature
   if you know it, what else was resident, anything unusual.
6. **Capture hardware, do not type it.** `uv run atlas-bench hwinfo --json`. Never copy
   specifications from a product page into a field the capture contradicts.
7. **Leave the fields CI owns alone.** `provenance.github_user_id`, `provenance.commit` and
   `provenance.pr` are `null` when you commit. CI and the build fill them in.
8. **Record the gotchas.** If you had to know something to make the run work — a flag whose
   default is a lie, a parser name that only resolves under one spelling, a container tag
   that exists only for aarch64 — put it in `gotchas[]`. That is the part of the run that
   outlives the number.

## The command sequence

```bash
# 1. get the repository and read this file
git clone https://github.com/0xBakeer/inference-atlas.git
cd inference-atlas

# 2. capture the hardware truthfully
uv run atlas-bench hwinfo --json

#    If the capture matches no hardware/*.json detect rule, STOP and add the hardware file
#    first, from the captured output. Say so in the PR.

# 3. install the engine at the pinned version and fetch the weights
docker pull vllm/vllm-openai:v0.27.1        # whatever the packet says
hf download Qwen/Qwen3.8-27B-FP8

# 4. serve with exactly the flags in the packet, and wait for health
vllm serve Qwen/Qwen3.8-27B-FP8 --max-model-len 262144 --gpu-memory-utilization 0.44 ...

# 5. run the workloads (task.json is the JSON packet)
uv run atlas-bench run --spec task.json

# 6. validate locally — the same code CI runs
pnpm install
pnpm validate

# 7. branch, commit, pull request
git checkout -b result/vllm-qwen-qwen3.8-27b-nvidia-gb10-dgx-spark-fa19e1
git add results/
git commit -m "results: vllm 0.27.1 Qwen/Qwen3.8-27B/fp8 on nvidia-gb10-dgx-spark"
git push -u origin result/vllm-qwen-qwen3.8-27b-nvidia-gb10-dgx-spark-fa19e1
gh pr create --base main \
  --title "results: vllm 0.27.1 Qwen/Qwen3.8-27B/fp8 on nvidia-gb10-dgx-spark" \
  --label results --body-file pr-body.md
```

**You open the pull request.** Every measurement runs on your own machine; CI only validates
the files and builds the site. Nothing in this repository benchmarks anything for you, so a
run that never becomes a PR never happened.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xBakeer/inference-atlas](https://github.com/0xBakeer/inference-atlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
