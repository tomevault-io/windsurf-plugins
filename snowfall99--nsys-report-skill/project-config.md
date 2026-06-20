---
trigger: always_on
description: Analyze an existing NVIDIA Nsight Systems (nsys) timeline report to find where time goes across an application — GPU idle/utilization, kernel & CUDA-API breakdown, memory transfers, stream concurrency, NVTX phases, and (multi-GPU) NCCL. Use when the user asks to read/analyze an nsys report (.nsys-rep / .qdrep / .sqlite), diagnose end-to-end GPU performance, find why training or inference is slow or the GPU is under-utilized, or check whether code is launch-bound or sync-bound — including Chinese
---


# Skill: Nsight Systems Report Analysis (RTX PRO 6000 / Blackwell)

**When to use:** the user hands you an `nsys` report (or points at one) and wants to know where time goes — *is the GPU busy? what dominates? why is end-to-end slow?* Triggers: "analyze this nsys report", "why is GPU utilization low", "is this launch-bound", "where is the time going", "为什么这个 pipeline 慢", "帮我看一下 nsys".

**Target hardware (this repo):** NVIDIA RTX PRO 6000 Blackwell Workstation Edition — chip GB202, `sm_120` (CC 12.0), 188 SMs, ~1792 GB/s GDDR7, 96 GB, 134 MB L2. Timeline analysis is almost entirely hardware-agnostic — only peak-bandwidth interpretation and GPU-metrics-set availability are device-specific, and those are marked. The workflow works on any GPU Nsight Systems supports. (`TARGET_INFO_GPU` in the SQLite export carries the exact device for whatever report you're handed — read it, don't assume.)

---

## Scope — read this first

This skill **analyzes a report that already exists**. It does **not** build harnesses, write kernels, or run the application. Your inputs are one of:

- `report.nsys-rep` — current format (open in the GUI, or process with the CLI)
- `report.qdrep` — legacy format (convert/export the same way)
- `report.sqlite` — already exported

If — and only if — a report is missing, malformed, or missing the data you need (e.g. no NVTX, no GPU metrics), tell the user the exact `nsys profile` command to (re)collect it (see [Appendix: collection recipes](#appendix-collection-recipes)). Then stop and wait for the report. **Do not** profile on the user's behalf.

---

## Golden rule

**Read the report → diagnose → propose, in that order. Let the timeline tell you where time goes. Never guess.**

System-level performance almost always has a single dominant cause that the report shows in seconds: the GPU is idle waiting on the CPU, or thousands of tiny kernels are launch-bound, or a blocking `cudaStreamSynchronize` serializes everything, or a pageable `cudaMemcpy` stalls the pipeline. Don't hypothesize before you have the numbers, and don't write a wall of generic suggestions — rank them by **measured impact** (how much wall-clock each would actually recover).

---

## Workflow

### Phase 0 — Set up a run directory
Create `profile/<analysis_name>/` (one analysis = one directory; don't mix runs). Put the report in it and write all derived files (`stats/`, the `.sqlite`, `REPORT.md`) alongside. Resolve the report path and `export REPORT=...` for the commands below.

### Phase 1 — Frame the question
Before touching the data, write down: what is this report of (training step? inference request? a kernel pipeline?), how long is the captured window, and what does the user actually want answered ("why is it slow" vs "is the GPU saturated" vs "where's the memcpy time"). The answer determines which dimension matters.

### Phase 2 — Run the expert-system rules **first**
```bash
nsys analyze "$REPORT"
```
Nsight Systems ships a rules engine that auto-flags the usual suspects — GPU starvation, gaps in GPU utilization, frequent synchronization, pageable-memory copies, tiny kernels, etc. **Read these before doing anything by hand**; they often point straight at the answer. Treat each as a lead to confirm with numbers, not as the conclusion.

### Phase 3 — Pull the canned summaries
```bash
nsys stats --report cuda_gpu_kern_sum,cuda_api_sum,cuda_gpu_mem_time_sum,nvtx_sum "$REPORT"
```
This gives you the top kernels (total time / count / avg duration), the CUDA API breakdown, memory-op time, and NVTX ranges. `nsys stats --help-reports` lists every available report. Save outputs under `profile/<analysis_name>/stats/`.

### Phase 4 — Export to SQLite for anything deeper
The canned reports can't tell you GPU **idle %**, timeline **gaps**, per-stream **concurrency**, or any custom join. Export and query:
```bash
nsys export --type sqlite -o "${REPORT%.nsys-rep}.sqlite" "$REPORT"
```
See [SQLite quick reference](#sqlite-quick-reference) for the schema and copy-pasteable queries.

### Phase 5 — Work through the six analysis dimensions
See [Analysis dimensions](#analysis-dimensions). All matter, but on any given report only 1–2 dominate. Compute the headline number for each (even roughly) so you can rank.

### Phase 6 — Diagnose and write the report
Match the observed pattern to the [Diagnosis playbook](#diagnosis-playbook), then write `profile/<analysis_name>/REPORT.md`: the setup, the headline metrics, per-dimension findings **with cited numbers**, and optimization directions **ranked by expected wall-clock recovery** with caveats. Specificity is the deliverable.

---

## Analysis dimensions

| # | Dimension | Headline question | Where to look |
|---|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Snowfall99/nsys-report-skill](https://github.com/Snowfall99/nsys-report-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
