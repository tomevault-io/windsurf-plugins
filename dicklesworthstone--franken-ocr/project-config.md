---
trigger: always_on
description: > Guidelines for AI coding agents working in this Rust codebase.
---

# AGENTS.md — franken_ocr

> Guidelines for AI coding agents working in this Rust codebase.

---

## RULE 0 — THE FUNDAMENTAL OVERRIDE PREROGATIVE

If I tell you to do something, even if it goes against what follows below, YOU MUST LISTEN TO ME. I AM IN CHARGE, NOT YOU.

---

## RULE NUMBER 1: NO FILE DELETION

**YOU ARE NEVER ALLOWED TO DELETE A FILE WITHOUT EXPRESS PERMISSION.** Even a new file that you yourself created, such as a test code file. You have a horrible track record of deleting critically important files or otherwise throwing away tons of expensive work. As a result, you have permanently lost any and all rights to determine that a file or folder should be deleted.

**YOU MUST ALWAYS ASK AND RECEIVE CLEAR, WRITTEN PERMISSION BEFORE EVER DELETING A FILE OR FOLDER OF ANY KIND.**

---

## Irreversible Git & Filesystem Actions — DO NOT EVER BREAK GLASS

1. **Absolutely forbidden commands:** `git reset --hard`, `git clean -fd`, `rm -rf`, or any command that can delete or overwrite code/data must never be run unless the user explicitly provides the exact command and states, in the same message, that they understand and want the irreversible consequences.
2. **No guessing:** If there is any uncertainty about what a command might delete or overwrite, stop immediately and ask the user for specific approval. "I think it's safe" is never acceptable.
3. **Safer alternatives first:** When cleanup or rollbacks are needed, request permission to use non-destructive options (`git status`, `git diff`, `git stash`, copying to backups) before ever considering a destructive command.
4. **Mandatory explicit plan:** Even after explicit user authorization, restate the command verbatim, list exactly what will be affected, and wait for a confirmation that your understanding is correct. Only then may you execute it.
5. **Document the confirmation:** When running any approved destructive command, record (in the session notes / final response) the exact user text that authorized it, the command actually run, and the execution time.

---

## Branch Policy

- Primary branch is `main`.
- Do not reference `master` in docs/scripts.
- If release instructions require sync, push `main:master` after `main`.

---

## Project Mission

`franken_ocr` is a **pure-Rust, memory-safe, CPU-hyper-optimized library + single-binary CLI (`focr`)** that runs the **Baidu Unlimited-OCR** vision-language document-parsing model **with no general ML framework**. We transform the model's bf16 weights into a custom quantized on-disk form (int8 first, int4 in refinement rounds) and write **model-specific kernels** whose only job is to run *this one model* as fast as possible on:

- **Apple Silicon / ARM64** — NEON, FEAT_DotProd (SDOT), FEAT_MATMUL_INT8 (SMMLA / i8mm)
- **Intel / AMD x86-64** — AVX2, AVX-VNNI, AVX-512-VNNI (and AMX tiles where present)

**CPU is the priority** (most target hosts lack a usable GPU); CUDA is an explicit later stretch goal.

It is built on:
- `/dp/frankentorch` (`ft-kernel-cpu`, `ft-core`, `ft-serialize`) — custom CPU tensor kernels, consumed at the **kernel** level (not the autograd level).
- `/dp/asupersync` — structured-concurrency runtime, for **orchestration / cancellation / IO only** (not for intra-op math parallelism).
- `/dp/frankensqlite` (`fsqlite`) — durable local run state + telemetry (NEVER `rusqlite`).

**The single source of truth for what we are building and why is [`COMPREHENSIVE_PLAN_FOR_FRANKEN_OCR.md`](COMPREHENSIVE_PLAN_FOR_FRANKEN_OCR.md).** Read it before writing any kernel.

### What this model is (one paragraph)

Unlimited-OCR is an **end-to-end VLM**, a DeepSeek-OCR derivative: a **DeepEncoder** vision tower (SAM-ViT-B → 16× conv token-compressor → CLIP-L/14) → a single **linear projector** (2048→1280) → a **DeepSeek-V2 MoE decoder** (12 layers, hidden 1280, 10 MHA heads — `use_mla=false`, 64 routed + 2 shared experts, top-6, vocab 129280) whose attention is replaced by **R-SWA** (Reference Sliding Window Attention, window 128), bounding generated-token KV while retaining all reference tokens. bf16, 6.67 GB single safetensors shard. **License: MIT (Copyright (c) 2026 Baidu)** — we may legally redistribute a quantized derivative if we ship that notice.

---

## Product Shape

The project must be both:
1. A reusable Rust library for embedding the OCR pipeline (`OcrEngine::recognize(...)`), **synchronous and blocking** — the async runtime is an owned implementation detail.
2. A standalone CLI binary `focr` with:
   - **robot mode** (agent-first, versioned NDJSON, self-describing `robot schema`)
   - human mode (`focr ocr <image>` → markdown, or `--json`)

Input: **document images only in v1** (PNG/JPG/…; PDF is rasterized out-of-band — see plan §7.7). No Python, no FFI, **no network at inference time**, no GPU required.

---

## Porting Workflow (Spec-First)

Implementation follows spec documents, not ad-hoc copying. Read in this order:
1. [`COMPREHENSIVE_PLAN_FOR_FRANKEN_OCR.md`](COMPREHENSIVE_PLAN_FOR_FRANKEN_OCR.md) — the master plan (architecture, kernel strategy, phased roadmap, verification methodology).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/franken_ocr](https://github.com/Dicklesworthstone/franken_ocr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
