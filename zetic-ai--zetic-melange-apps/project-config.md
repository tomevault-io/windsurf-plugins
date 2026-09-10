---
trigger: always_on
description: This file is shared context for every session in this repo. Read it fully before acting.
---

# CLAUDE.md — ZETIC On-Device ML Demo Apps

This file is shared context for every session in this repo. Read it fully before acting.

**The doc set:** CLAUDE.md (this file — shared context + SDK realities) · EXPLORATION.md (Stage 0: find + export a model) · AGENTS.md (orchestration protocol + gates) · VALIDATION.md (the quality battery). Read all four before running the workflow.

---

## 1. What this repo is

A collection of small, polished Flutter demo apps, each wrapping one on-device ML model deployed through ZETIC's Melange SDK. The apps exist to support ZETIC's go-to-market motion (VivaTech leads, industrial and automotive prospects). Each app should look trade-show ready, not prototype-grade.

Every app follows the same anatomy: one Melange model, a Flutter UI, and a pure-Dart pre/post-processing pipeline. The first reference implementation is PyroGuard (YOLO11s fire and smoke detector).

---

## 2. ZETIC / Melange in one paragraph

Melange automatically compiles a trained ONNX model into NPU-optimized binaries, benchmarks them across a real device farm, and serves each end-user device the best-performing binary at runtime. The developer uploads an ONNX plus a sample input on the dashboard, waits for the CONVERTING then OPTIMIZING then READY cycle, then pulls the model into the app with the SDK. NPU acceleration is the whole point: on flagship silicon, inference can drop from hundreds of CPU milliseconds to single-digit NPU milliseconds.

That is the ideal. In practice (see section 5) "best-performing" is not guaranteed: selection can serve a crashing artifact, or fall back to CPU rather than the NPU, and a benchmarked NPU row may never be served for a given chip. Treat the served artifact on the device console — not the dashboard's headline number — as ground truth, and budget for a CPU-speed fallback until the NPU path is confirmed on hardware.

---

## 3. Roles

**Human (Ajay).** Owns everything the agents cannot do: the **Melange dashboard upload and model creation** (GATE 0 — the one manual step in the pipeline), physical-device runs on the iPhone, secrets (the personal key at build time), and merges/PR approval. Routine gate approvals (GATE 1 spec, GATE 2 approach, GATE 3 acceptance) are held by the orchestrator and reported after the fact — see AGENTS.md, "Gate ownership". The human is the only one who sees real on-device behavior. (ONNX export and per-app folder creation are now done by the Explorer in Stage 0; the human just drags the artifacts into the dashboard.)

**Master orchestrator (Opus 4.8, maximum reasoning effort).** Does not write app code. Its job is to run Stage 0 exploration (spin off Explorers, see EXPLORATION.md), turn each chosen model into a complete, gap-free per-app spec (template in section 6), then delegate to one worker per app, hold the checkpoint gates (reviewing and self-approving GATE 1/2 and accepting GATE 3 itself, interrupting the human only per AGENTS.md "Gate ownership"), and review what each agent returns. Workers start building in parallel with the GATE-0 upload, against late-binding model constants (section 4). It delegates **all** task-specific work (sourcing, edits, builds, research, validation, even doc edits and quick one-line checks) to agents and does zero concrete work inline — its own job is only to decide, delegate, review, and hold gates. Every agent it deploys (Explorer, worker, or one-off) runs in its own dedicated git worktree from the moment it is deployed; the orchestrator sets that worktree up as it delegates. Delegation is manual and deliberate, never automatic. See AGENTS.md.

**Explorer agent (Opus 4.8, high reasoning effort), one per use-case.** Stage 0 only. Searches Hugging Face for its assigned use-case, reasons which model is best for Melange, exports it to ONNX, generates the sample input, and populates the app folder (artifacts + `melange_upload.md` + `model_selection.md` + a pre-drafted spec stub). Does not write app code. Stops at GATE 0 and tells the human exactly what to upload and paste back. See EXPLORATION.md.

**Worker agent (Opus 4.8, high reasoning effort), one per app.** Owns a single app on its own git branch and its own worktree (like every agent, set up at deployment). Tries to one-shot the app from the spec, runs the full validation loop against VALIDATION.md on its own, and stops at the defined gates rather than dumping unverified work. A worker may run its validation loop as a subroutine (write, test, read failures, fix, repeat) but does not silently push past a gate.

---

## 4. Standard app anatomy

```
<AppName>/                     # the app folder (created by the Explorer in Stage 0)
  export.py                    # Stage 0: re-runnable export recipe
  <model>.onnx                 # Stage 0: exported artifact (drag into dashboard)
  sample_input.npy             # Stage 0: sample input (drag into dashboard)
  melange_upload.md            # Stage 0: the human's GATE-0 dashboard instructions
  model_selection.md           # Stage 0: top-5 shortlist + winner rationale
  HANDOFF.md                   # living Jira ticket — created first (after GATE 1), finalized at GATE 3
  Flutter/                     # the app itself (worker-owned, built in parallel

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zetic-ai/ZETIC_Melange_apps](https://github.com/zetic-ai/ZETIC_Melange_apps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
