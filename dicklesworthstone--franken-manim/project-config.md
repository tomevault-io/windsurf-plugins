---
trigger: always_on
description: > Guidelines for AI coding agents working in this Rust codebase.
---

# AGENTS.md — franken_manim

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

`franken_manim` is a **sovereign, deterministic, programmatic mathematical-animation engine in pure Rust** — a ground-up rebuild of manim (Grant Sanderson's engine behind 3Blue1Brown) on the Dicklesworthstone FrankenSuite. Its native Rust library, CLI (`fmn`), and live Studio ship without CPython. The separately installed `fmn-python` PyO3 portal presents the `manimlib` surface inside a supported host CPython so existing manim scene code runs source-unedited.

The contract is **API compatibility and semantic fidelity — deliberately not output identity**. Under manim's familiar names, FrankenManim does the *correct* thing: `MoveAlongPath` moves at true constant speed, `get_arc_length` returns the arc length, colors composite in a defined color model, the clock never drifts. The Reference — `3b1b/manim` @ `6199a00d4c1b1127ebe45cb629c3f22538b10e13` — is a design oracle and an aesthetic bar, **never a pixel warden**. Every deliberate divergence is a documented **Behavior Note** (§16.8).

The leapfrogs, in brief:

- **One-binary native installation.** The Rust API, standalone `fmn`, and Studio need no LaTeX, dvisvgm, Pango, fontconfig, system fonts, or CPython. Typesetting — text *and* TeX-style mathematics — is native, built on bundled fonts (Computer Modern included) and **fmd-math**, a clean-room TeX-math layout engine that lands as workspace crates in franken_markdown. The optional `fmn-python` wheel is outside this one-binary claim and requires a supported host CPython plus NumPy.
- **One external tool.** `ffmpeg`, sandboxed, optional (native y4m/PNG-sequence/GIF outputs exist), used only for video encode/mux/transcode. There is no second tool and no carve-out for one. The standalone engine never locates or launches CPython; the optional Python portal is imported by its host interpreter.
- **Certified determinism.** `--reproducible` yields bit-identical raw frames, canonical PNGs, and WAV across the certified platform matrix from a content-hashed input closure. Encoded video is equivalence-classed, never bit-promised.
- **Pinned bits, free scheduler.** One semantic renderer (Lumen) over multiple execution engines — certified CPU, fast CPU (SIMD tiers), and a standard-only Accelerator Annex (Metal/CUDA via frankentorch) — where every schedule reproduces certified bits exactly and everything that can't is quarantined to `standard` and labeled.
- **Farm-class scaling from a single scene.** Pure-segment frame parallelism, pipelined frame stages, and topology-aware render teams saturate a 96-core workstation while certified output stays identical at any thread count.

**The single source of truth for what we are building and why is [`COMPREHENSIVE_PLAN_FOR_THE_DESIGN_OF_FRANKEN_MANIM.md`](docs/planning/COMPREHENSIVE_PLAN_FOR_THE_DESIGN_OF_FRANKEN_MANIM.md)** (Revision 4). Read it before writing any subsystem.

### What we stand on (the FrankenSuite substrate)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/franken_manim](https://github.com/Dicklesworthstone/franken_manim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
