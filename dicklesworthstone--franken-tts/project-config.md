---
trigger: always_on
description: > Guidelines for AI coding agents working in this Rust codebase.
---

# AGENTS.md — franken_tts

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

`franken_tts` is a **pure-Rust, memory-safe, CPU-hyper-optimized library + single-binary CLI (`ftts`)** that runs the **Qwen3-TTS-12Hz-0.6B-Base** zero-shot voice-cloning text-to-speech model **with no general ML framework**. The thesis: **turn the model's hidden 15-step residual-code microdecoder from its largest CPU liability into its largest optimization advantage** (cache-resident hot-packing, per-depth quantization, speculative block verification). We transform the bf16 weights into a canonical quantized artifact (int8 first; int4 tried **on the microdecoder before the talker**, and only after it passes both the per-ISA speed test and the blind-listening equivalence test) and write **model-specific kernels** whose only job is to run *this one model* as fast as possible on:

- **Apple Silicon / ARM64** — NEON, FEAT_DotProd (SDOT), FEAT_MATMUL_INT8 (SMMLA / i8mm)
- **Intel / AMD x86-64** — AVX2, AVX-VNNI, AVX-512-VNNI (and AMX tiles where present)

**CPU is the priority** (most target hosts lack a usable GPU; how far past real time each SKU can go is set honestly by the plan's Phase −1B cost model — the first-order traffic floor is ≈20.7 GB/s at 1× real time, so no blanket speed promises). An early **Metal feasibility spike for the microdecoder** runs in Phase −1B; Metal *productization* is the Phase-6 stretch; CUDA sits behind even that.

**Kyutai Pocket TTS 100M is the mandatory challenger / ultra-edge second model** — governed by the three-gate bakeoff in the plan (§11 there: upstream quality → architectural systems potential → optimized confirmation). Qwen is the quality-first champion unless all three gates say otherwise.

It is built on:
- `/dp/frankentorch` (`ft-kernel-cpu`, `ft-core`, `ft-serialize`) — custom CPU tensor kernels, consumed at the **kernel** level (not the autograd level), through the single facade in `ftts-kernels`.
- `../asupersync` — structured-concurrency runtime, for **orchestration / cancellation / streaming IO only**. The hot decode loop runs on our own fixed **`KernelTeam`** (persistent workers, static partitions, no work-stealing); rayon is retained only for the f32 port, converters, enrollment, and as the incumbent to beat.
- The CLI is **stateless by default** (no persisted synthesis history — texts and voices are sensitive). If durable opt-in traces/state are enabled, they use `/dp/frankensqlite` (`fsqlite`) — NEVER `rusqlite`.

**The single source of truth for what we are building and why is [`COMPREHENSIVE_PLAN_FOR_FRANKEN_TTS.md`](COMPREHENSIVE_PLAN_FOR_FRANKEN_TTS.md).** Read it before writing any kernel. The governing methodology skill is `/ai-model-into-rust-mega-fused-hyper-kernel`.

### What this model is (one paragraph — read the nested loop carefully)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/franken_tts](https://github.com/Dicklesworthstone/franken_tts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
